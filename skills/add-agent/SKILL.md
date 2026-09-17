---
name: add-agent
description: "Add a new agent to this kit — describe it, or import an existing agent file from anywhere. Analyzes the current agent pipeline, proposes where the new agent belongs, writes it to the kit's conventions on a new branch, and wires it into the workflows, the adopt skill, the README and the version. Triggers on: /mobile-kit:add-agent [description or path]"
disable-model-invocation: true
---

You are now the agent integrator. You add a new agent to this kit and wire it in so it is actually spawned.

Work directly — do not delegate the writing. You may spawn Explore agents for the analysis in Step 2.

Ask with the **AskUserQuestion** tool every time. Never ask in plain prose — that ends the turn and looks like the skill stopped. The tool always adds a free-text answer of its own, which is how the user types a description, a spot, a path or a correction.

## Before you start

Unlike every other skill in this kit, you do NOT need `.claude/docs/PROJECT_CONTEXT.md`. You work on agent definitions, not on product code.

Never write into the installed plugin cache (`~/.claude/plugins/cache/...`) — it is overwritten on every update. Everything is written to the plugin repository located in Step 4, on a new branch. Until then, read only.

---

**1. ASK — What does the new agent do?**
Use the AskUserQuestion tool to ask the user:
- "Add an existing agent"
- "Suggest what is missing"
- the user types the agents description

If the answer was "Add an existing agent" ask the user where to find it. Then read it to get an
idea what it does.

If the answer was "Suggest what is missing", do Step 2's reading first, then offer the gaps you found as choices.

**2. ANALYZE — Where to locate the new agent?**
Read the existing agents, the skills, the pipeline. Look for all fitting spots and find the best.

Note the `##` headings already taken in the shared context file, and check honestly whether an existing agent already does this job.

**3. ASK — Where to locate the new agent?**
Use the AskUserQuestion tool to ask the user:
- List of spots, each with a reason, one recommended.
- the user types the spot

Add "Extend `<existing agent>` instead" when Step 2 showed the job is already covered. If you concluded the agent should not exist, say so and recommend that option. The user decides; if they overrule you, build it in full without arguing again.

**4. ASK — Where is the plugins repository?**
The mobile-kit plugins repository is needed to create a new branch for the changes.
Use the AskUserQuestion tool to ask the user:
- "create a fork for me"
- the user types the location of the repository

Look for clones on disk first and offer them as choices. "create a fork for me" runs `gh repo fork <upstream> --clone`; if `gh` is not authenticated, stop and ask the user to log in. Verify the path: it must hold `.claude-plugin/plugin.json` and `agents/`, and the working tree must be clean. Then create the branch.

**5. SHOW — Preview.**
Show every decision and the files that will be edited. then ask.
Use the AskUserQuestion tool to ask the user:
- "go"
- "cancel"
- the user types adjustments

Decide everything not asked yourself — name, boundary, tools, findings blocking or advisory, memory directory, `##` heading, model — and show all of it here. This is the only place the user sees those decisions. Write nothing before "go".

If the agent needs project facts `PROJECT_CONTEXT.md` does not carry, warn separately: it forces a `CONTRACT_VERSION` bump and a re-adopt in every project.

**6. WRITE — Write the agent file.**
Copy the structure of the existing agents, so it reads the project context, has a memory,
writes to the log, and ends with the phrase the workflow waits for.

Do not copy the memory directory, the `##` heading or the name: memory is `.claude/agent-memory/mobile-kit-<name>/`, the heading must be unused, and the name must not collide with an agent in this kit, another plugin, or a built-in.

**7. WRITE — Adjust the other files.**
like Skills, adopt, README, version.

For the workflow skills: the step, the spawn prompt (context file path, what to read, what to produce, which heading to append under), the gate the orchestrator checks afterwards, and the final report row. `orchestrate` and `bug-hunt` each have a full and a fast step list.

**8. REPORT — What changed, what is still open.**
User pushes the repo, then everyone runs
`/plugin marketplace update` and `claude plugin update`.

List every file changed and every touchpoint as done, skipped with a reason, or needing a decision. Say plainly whether the agent has been spawned for real yet — until a workflow has spawned it once, the integration is unproven.

## Rules

- Ask with the AskUserQuestion tool. Never end a turn on a bare question.
- Ask the questions in order. Do not guess an answer the user has not given.
- Everything not asked is your decision, and all of it is shown in the preview.
- Write nothing before the user says "go".
- Never write into the installed plugin cache.
- Never commit, never push, never bump a version silently.
- An agent that no workflow spawns is not integrated. Say so if that is the outcome.
