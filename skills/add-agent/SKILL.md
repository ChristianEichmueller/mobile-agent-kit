---
name: add-agent
description: "Add a new agent to this kit — describe it, or import an existing agent file from anywhere. Analyzes the current agent pipeline, proposes where the new agent belongs, writes it to the kit's conventions, and wires it into the workflows, the adopt skill, the README and the version. Triggers on: /mobile-kit-test:add-agent [description or path]"
disable-model-invocation: true
---

You are now the agent integrator. You add a new agent to this kit and wire it in so it is actually spawned.

Follow the steps below exactly. You work directly — do not delegate the writing, but you may spawn Explore agents for the analysis in Step 2.

## Before you start

Unlike every other skill in this kit, you do NOT need `.claude/docs/PROJECT_CONTEXT.md`. You work on agent definitions, not on product code.

Find your target instead:

- **Plugin agents** live in the plugin repository — a directory with `.claude-plugin/plugin.json` and `agents/`.
- **Project agents** live in `<project>/.claude/agents/`.

Never write into the installed plugin cache (`~/.claude/plugins/cache/...`). It is overwritten on every update and keyed by version. If that is the only copy available, say so and continue with Step 4.

The writing target is decided in Step 4. Until then, read only.

## Step 1: ASK — What does the new agent do?

Ask what the new agent should do. The answer is free text: a description, or a path to an existing agent file. Work out which you got.

If the user already passed one as an argument, skip the question and confirm what you understood.

For a path: read the file now. Keep its role — you will rewrite everything around it in Step 6.

---

## Step 2: ANALYZE — Where to locate the new agent?

Read, before proposing anything:

- every `agents/*.md` in this kit — at minimum the frontmatter, so you know each agent's job and boundary
- every `skills/*/SKILL.md` — the workflow steps, which agent runs where, what is parallel, what is sequential
- the `##` headings already used in the shared context file, so you know which are taken

Then work out **all** spots where the new agent could fit, and pick the best. A spot is: which workflow, at which step, before or after which agent, parallel or sequential.

Also check honestly whether an existing agent already does this job. `tech-lead`, `qa-reviewer` and `code-optimizer` overlap already — a fourth reviewer needs a boundary you can state in one sentence.

---

## Step 3: ASK — Where to locate the new agent?

Offer the spots you found as choices, each with a one-line reason, one recommended.

Add **"Extend `<existing agent>` instead"** when Step 2 showed the job is already covered.

If you concluded the agent should not exist, say so plainly and recommend that option. The user decides. If they overrule you, build it in full without arguing again.

---

## Step 4: ASK — Who will use the agent?

Ask, with these answers:

- **"Only this project"** — local file, no release
- **"The whole team"** — goes into the plugin, needs a version bump

If the answer is "the whole team", you need a copy of the plugin repo — never the installed cache.

Look for a clone on disk, then ask which to use: the clones you found, or **"Create a fork for me"** (`gh repo fork <upstream> --clone`; if `gh` is not authenticated, stop and ask the user to log in).

Verify whichever path you end up with: it must hold `.claude-plugin/plugin.json` and `agents/`, and the working tree must be clean.

This answer decides the memory directory in Step 6, so do not skip it.

---

## Step 5: SHOW — Preview

Show every decision and every file that will be edited. Decide these yourself — do not ask:

```
Agent:     <name>
Job:       <one line>
Boundary:  not <agent A> (which does X), not <agent B> (which does Y)
Spot:      <workflow, step, parallel or sequential>
Scope:     only this project | the whole team
Tools:     read-only | full
Findings:  advisory | blocking
Memory:    <exact directory>
Heading:   ## <Heading>
Model:     opus

Files:
  + <the agent file>
  ~ <each workflow, adopt, README, plugin.json>
```

If the agent needs project facts `PROJECT_CONTEXT.md` does not carry yet, warn separately here: it forces a `CONTRACT_VERSION` bump and a re-`adopt` in every project already using this kit.

Then ask: **go** or **cancel**. Anything else the user says is a correction — apply it and show the preview again.

Write nothing before "go".

---

## Step 6: WRITE — Write the agent file

Copy the structure of the existing agents, so the new one reads the project context, has a memory, writes to the log, and ends with the phrase the workflow waits for.

Three things must not be copied:

- **the memory directory** — `.claude/agent-memory/mobile-kit-test-<name>/` for the whole team, `.claude/agent-memory/<name>/` for one project only. The wrong one means memory nothing reads.
- **the `##` heading** — it must be unused. Grep the skills and agents first.
- **the name** — it must not collide with an agent in this kit, another plugin, or a built-in (`Explore`, `Plan`, `general-purpose`). The filename stem must equal `name:`.

For an imported file: keep the role body, replace everything around it, and note what you changed.

---

## Step 7: WRITE — Adjust the other files

Whatever the spot from Step 3 requires:

- **the workflow skills** — the step, the spawn prompt (context file path, what to read, what to produce, which heading to append under), the gate the orchestrator checks afterwards, the row in the final report table, and `## Rules` if the agent is mandatory. `orchestrate` and `bug-hunt` each have a full and a fast step list — state which ones the agent joins.
- **`skills/adopt/SKILL.md`** — add the memory directory to Step 4, so it is scaffolded on adoption.
- **`README.md`** — the agent table, and the skills table if you added a workflow.
- **`.claude-plugin/plugin.json`** — bump `version`. Without the bump nobody ever receives the agent.

The last two apply to the plugin scope only. A project-local agent needs none of them.

---

## Step 8: REPORT — What changed, what is still open

List every file you changed, and every touchpoint as done, skipped with a reason, or needing a decision. Never skip one silently.

- **Only this project:** done — the agent works now. Tell the user to run `/reload-plugins`.
- **The whole team:** the user pushes the repo, then everyone runs `/plugin marketplace update` and `claude plugin update`.

Say plainly whether the agent has been spawned for real yet. Until a workflow has spawned it once and it has appended its section, the integration is unproven.

Never commit and never push — the user reviews and does that.

## Rules

- Ask the four questions in order. Do not guess an answer the user has not given.
- Everything not asked is your decision, and all of it is shown in the preview.
- Write nothing before the user says "go".
- Never write into the installed plugin cache.
- Never commit, never push, never bump a version silently.
- An agent that no workflow spawns is not integrated. Say so if that is the outcome.
