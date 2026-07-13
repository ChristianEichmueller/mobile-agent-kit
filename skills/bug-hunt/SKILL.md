---
name: bug-hunt
description: "Diagnose a bug in a mobile project, write a failing repro test that pins it down, fix the code, then verify with review. TDD-style bug workflow: repro test first, then fix, then confirm test flips green. Triggers on: /mobile-kit:bug-hunt <bug description or stacktrace>"
---

You are now the orchestrator for a bug hunt. Follow the steps below exactly. Coordinate by spawning specialized agents and reading a shared context file between steps. Do NOT delegate orchestration — YOU execute these steps directly.

## When To Use This Skill

Use `/mobile-kit:bug-hunt` for:
- A crash report / stacktrace / crash-reporting-tool issue
- A user-reported bug ("X doesn't work when I do Y")
- A regression discovered manually or via CI
- A rotation / lifecycle / edge-case bug the user found

Do NOT use for:
- Adding a new feature — use `/mobile-kit:orchestrate` instead
- Refactoring without a specific bug — use `/mobile-kit:orchestrate` instead
- Code review of existing changes — use `/mobile-kit:review-loop`

## Setup

Precondition: the project must contain `.claude/docs/PROJECT_CONTEXT.md`. If missing, stop and tell the user to run `/mobile-kit:adopt` first.

1. Generate a timestamp: run `date '+%Y-%m-%d-%H-%M-%S'` via Bash
2. Derive a short kebab-case name from the bug (e.g., "caption-lost-rotation", "npe-on-empty-list")
3. Create the shared context file at `.claude/agents-log/<timestamp>-bug-<short-name>.md` with this initial content:

```markdown
# Bug Hunt: <short bug description>

## Bug Report
<paste the user's full bug description, stacktrace, or repro steps here>

## File References
<any file paths, screenshots, or logs the user provided>
```

Store the context file path — pass it to every agent.

## Workflow

Execute in order. Read the context file between steps to verify progress.

### Step 1: Bug Fixer — Diagnose ONLY (do not fix yet)

Spawn the bug fixer for root-cause analysis:
```
Agent(subagent_type: "bug-fixer")
```

Prompt must include:
- The bug description / stacktrace / repro steps
- Instruction: "READ the context file at `<path>`. Your ONLY task in this step is diagnosis — do NOT modify code. Read the relevant stacktrace or repro steps. Trace the root cause through the codebase with file:line evidence. If you need runtime evidence (logs, dispatcher state, view-model state) to be certain of the root cause, install TEMPORARY diagnostic prints, run the failing scenario, capture logs, then REMOVE the prints before finishing. When done, append your results under a `## Bug Fixer — Diagnosis` heading including: (a) confirmed root cause with file:line, (b) the exact user-observable symptom, (c) the diverging axis (why does it happen in scenario X but not Y), (d) a proposed fix approach (not code — just the approach)."

After it completes, read the context file. Verify:
- Root cause has concrete file:line references
- Symptom is described in user-observable terms
- No temporary diagnostic prints were left in prod code (grep confirms)

If the bug fixer's diagnosis is uncertain and asks a question, relay it to the user and wait.

### Step 2: Test Writer — Failing Repro

Spawn the test writer to pin down the bug with a failing test:
```
Agent(subagent_type: "test-writer")
```

Prompt must include:
- Instruction: "READ the context file at `<path>` — pay close attention to `## Bug Fixer — Diagnosis`. Write ONE OR MORE tests that reproduce this bug, using the project's UI/integration tests as defined by the test policy in `.claude/docs/PROJECT_CONTEXT.md`. Test names should read as the bug behavior (e.g., `captionIsLostAfterRotationAndSystemBackFromPreview`). Include a CONTROL test that documents the working path (e.g., `captionSurvivesReturnFromPreviewWithoutRotation`) — this pins down what the fix must NOT break. If the bug involves state persistence across a lifecycle event (rotation/orientation change, process death, background), add a SECOND-EVENT assertion (e.g., rotate twice) to guard against 'double-fire' fixes. Run the tests with the test commands from the project context and confirm the repro test FAILS and any control test PASSES. Add any missing helpers using the project's UI-test abstractions and fixture scenarios per the project context. When done, append under `## Test Writer — Repro`: the test names, files touched, test run results (showing repro=FAILED, control=PASSED)."

After it completes, verify:
- The repro test FAILS as expected — proves the bug is real
- The control test PASSES — proves the surrounding path still works
- Second-event assertion is present if lifecycle is involved

### Step 3: Bug Fixer — Fix

Spawn the bug fixer to apply the fix:
```
Agent(subagent_type: "bug-fixer")
```

Prompt must include:
- Instruction: "READ the context file at `<path>`. Your diagnosis is in `## Bug Fixer — Diagnosis`. The failing repro test(s) are in `## Test Writer — Repro`. Implement the smallest correct fix that makes the repro test(s) green WITHOUT breaking the control test(s) or any pre-existing tests. Do NOT modify the tests. Run the repro + control tests after fixing and report the outcome. When done, append under `## Bug Fixer — Fix`: files modified, the smallest possible diff summary, test run results (repro=PASSED, control=PASSED, plus any related regression tests you ran)."

After it completes, verify:
- Repro test transitions from FAILED to PASSED
- Control test stays PASSED
- Test files (in the project's test location(s) per the project context) were NOT modified — this is a critical check. If the bug fixer modified tests to make them green, the fix is invalid.

Two branches:

- **Repro PASSED + control PASSED + no test files touched** → proceed to Step 5.
- **Bug fixer believes a test is wrong** → the bug fixer must append `## Developer Test Concern` (yes, same heading as in `/mobile-kit:orchestrate`) with rationale. Proceed to Step 4.

### Step 4: Test Writer — Push-Back Loop (max 2 iterations)

Same push-back mechanism as `/mobile-kit:orchestrate`. Track iteration count.

Spawn test writer:
```
Agent(subagent_type: "test-writer")
```

Prompt: "Read the context file at `<path>`. The bug fixer has raised a concern about the repro test under `## Developer Test Concern`. Evaluate it honestly. Either adjust the repro test (append `## Test Writer — Iteration <n>` with reasoning) or defend it (append `## Test Writer — Defense (Iteration <n>)` with (i) the user-observable contract the bug violates, (ii) why the fixer's alternative is incomplete or bypasses the bug, (iii) a concrete example scenario). Re-run affected tests and report."

Branches:
- **Test adjusted** → re-spawn bug fixer to re-fix against the adjusted test. Return to Step 3 with iteration +1.
- **Test defended and bug fixer agrees (tests all green after refix)** → proceed to Step 5.
- **Iteration 2 unresolved** → STOP. Escalate to user via `## Escalation to User` summarizing both positions. Wait for user input.

### Step 5: Tech Lead + QA Reviewer (PARALLEL)

Spawn BOTH in parallel (single message, two tool calls):

**Tech Lead:**
```
Agent(subagent_type: "tech-lead")
```
Prompt: "Read the context file at `<path>` for full bug context, repro tests, and the fix. Review the fix for: (a) is this the smallest correct diff, (b) does it introduce new architectural violations, (c) are there other sites in the codebase with the SAME bug pattern that should be fixed in the same PR, (d) did the fixer modify tests (should be zero). Append `## Tech Lead Review` with findings and file:line references."

**QA Reviewer:**
```
Agent(subagent_type: "qa-reviewer")
```
Prompt: "Read the context file at `<path>` for full bug context, repro tests, and the fix. Look for edge cases the fix does not cover: null / empty / concurrent / lifecycle-transition / different device configurations. Verify the repro test's assertions actually pin the bug down (not just accidentally green). Suggest additional test cases if coverage has gaps. Append `## QA Review` with findings."

After BOTH complete, read the context file.

### Step 6: Fix Issues (IF NEEDED)

If tech-lead flags "other sites have this bug pattern" → decide with user before fixing more sites (scope creep guard).

If QA flags missing coverage:
1. Spawn test writer to add tests: "Read `<path>`. QA identified missing coverage under `## QA Review`. Add tests. Run them. Append under `## Test Writer — Coverage Extension`."
2. If new tests fail against current fix → spawn bug fixer to close the gap.

If tech-lead or QA flag correctness issues in the fix itself:
1. Spawn bug fixer to address them.
2. Re-run relevant reviewers.
3. Repeat until clean.

### Step 7: Design Guardian (CONDITIONAL)

Skip if zero UI files touched. Otherwise:
```
Agent(subagent_type: "design-system-guardian")
```
Prompt: "Read `<path>`. Review UI changes for design system compliance. Append `## Design Guardian Review`."

### Step 8: Final Report

Read the context file one final time. Present to user:

```
## Bug Hunt Complete

### Bug Summary
- **Symptom:** <one line>
- **Root cause:** <one line — file:line>
- **Fix:** <one line — file:line>

### Delegation Report
| # | Agent (subagent_type) | Agent ID | Status | Summary |
|---|----------------------|----------|--------|---------|
| 1 | bug-fixer diagnosis (bug-fixer) | <id> | completed | <one line> |
| 2 | test-writer repro (test-writer) | <id> | completed | <one line — N tests written, repro=RED, control=GREEN> |
| 3 | bug-fixer fix (bug-fixer) | <id> | completed | <one line — repro flipped to GREEN> |
| 4 | test-writer re-eval (test-writer) | <id> | completed/skipped | <only if push-back triggered> |
| 5 | tech-lead (tech-lead) | <id> | completed | <one line> |
| 6 | qa-reviewer (qa-reviewer) | <id> | completed | <one line> |
| ... | ... | ... | ... | ... |

**Repro tests:** <count> — all currently GREEN
**Push-back iterations:** <0 | 1 | 2 | escalated>
**Other sites with same pattern:** <N flagged by tech-lead — deferred to user | fixed in-scope>
**Context file:** `<path>`
```

Rules for the report:
- List EVERY agent spawn in chronological order (including re-runs and push-back)
- Agent ID is the ID returned by each Agent tool call
- MUST contain at minimum: 2x bug-fixer (diagnosis + fix), 1x test-writer, 1x tech-lead, 1x qa-reviewer

## Rules

- You MUST spawn bug-fixer BEFORE test-writer for diagnosis — the test needs the root cause to be pinned first
- You MUST spawn test-writer BEFORE the fix — the test is the contract
- The fix agent MUST NOT modify test files. If they do, the fix is invalid — re-spawn with correction instruction.
- Push-back loop capped at 2 iterations. Escalate on iteration 3.
- Tech-lead + QA are NEVER optional.
- Never commit — user reviews and commits manually.
- Never skip the final report.
- If the bug fixer requires temporary diagnostic prints in prod code for evidence, they MUST remove them before finishing. Grep for their tag as a sanity check.
