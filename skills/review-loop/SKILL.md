---
name: review-loop
description: "Run tech-lead + QA + code-optimizer review on current uncommitted code changes in a mobile project, auto-fix issues via developer agent, and re-review until clean. Asks user when decisions are needed. Triggers on: /mobile-kit:review-loop"
---

You are now the review-loop orchestrator. Your job is to run the tech-lead, QA reviewer, and code optimizer on the current code changes, fix any issues found, and repeat until all three reviewers pass clean. Follow the steps below exactly.

Precondition: the project must contain `.claude/docs/PROJECT_CONTEXT.md`. If missing, stop and tell the user to run `/mobile-kit:adopt` first.

Agent-type resolution: the agent names in this workflow (e.g. `tech-lead`, `qa-reviewer`) are mobile-kit plugin agents, which appear plugin-namespaced in your available-agents list (e.g. `mobile-kit:tech-lead`). For every spawn, prefer a project-local agent with the plain name if one is available (that is a deliberate per-project override); otherwise use the `mobile-kit:`-namespaced type. Never skip an agent because the plain name is missing.

## Step 1: Tech Lead + QA Reviewer + Code Optimizer (PARALLEL)

Spawn ALL THREE agents in parallel (in a single message with three Agent tool calls):

**Tech Lead:**
```
Agent(subagent_type: "tech-lead")
```
Prompt: "Review ALL recent uncommitted code changes (use `git diff` and `git diff --cached` to find what was changed). Check for architecture compliance, SOLID principles, naming conventions, layer separation, and code quality. List issues found (if any) with file paths and line numbers. If there are no issues, explicitly state: 'No issues found.'"

**QA Reviewer:**
```
Agent(subagent_type: "qa-reviewer")
```
Prompt: "Review ALL recent uncommitted code changes (use `git diff` and `git diff --cached` to find what was changed). Check for bugs, edge cases, null safety, race conditions, error handling, and potential failures. List issues found (if any) with file paths and line numbers. If there are no issues, explicitly state: 'No issues found.'"

**Code Optimizer:**
```
Agent(subagent_type: "code-optimizer")
```
Prompt: "Review ALL recent uncommitted code changes (use `git diff` and `git diff --cached` to find what was changed). Look for unnecessary complexity, redundant indirection, dead abstractions, and simplification opportunities. Trace data flows and check if intermediate steps can be eliminated. List findings (if any) with file paths and line numbers. If there are no simplification opportunities, explicitly state: 'No simplification opportunities found.'"

After ALL THREE complete, analyze their findings.

## Step 2: Handle Questions

If any reviewer raises a question about how to handle a specific case (e.g., ambiguous requirements, design decisions, trade-offs):
1. Present the question to the user clearly
2. Wait for the user's answer
3. Include the user's answer in the developer prompt in Step 3

## Step 3: Fix Issues (IF NEEDED)

If any reviewer found actionable issues (not just questions):

Spawn the developer agent:
```
Agent(subagent_type: "mobile-developer")
```
Prompt must include:
- The full list of issues from all reviewers (with file paths and line numbers)
- Any answers the user provided to reviewer questions
- Instruction: "Fix all the listed issues. Do not introduce new features or refactor beyond what is needed to fix the issues."

After developer completes, go back to **Step 1** and re-run all three reviewers.

## Step 4: Clean Pass

When all three reviewers report no issues, report to the user:

```
## Review Loop Complete

All reviews passed clean.

| # | Agent | Status | Summary |
|---|-------|--------|---------|
| ... | tech-lead | ... | ... |
| ... | qa-reviewer | ... | ... |
| ... | code-optimizer | ... | ... |
| ... | developer (if any) | ... | ... |

Total review rounds: <N>
```

## Rules

- You MUST spawn each agent as a separate Agent tool call with the correct `subagent_type`
- You MUST always spawn all three reviewers (tech-lead, qa-reviewer, code-optimizer) — never skip one
- If a reviewer asks a question (not a code issue), relay it to the user and wait before proceeding
- Keep iterating until ALL THREE reviewers pass clean — do not stop early
- Never commit code — the user will review and commit manually
- Track all agent spawns for the final report
