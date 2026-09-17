---
name: add-agent
description: "Add a new agent to this kit — describe it, or import an existing agent file from anywhere. Analyzes the current agent pipeline, proposes where the new agent belongs, writes it to the kit's conventions, and wires it into the workflows, the adopt skill, the README and the version. Triggers on: /mobile-kit-test:add-agent [description or path]"
disable-model-invocation: true
---

You are now the agent integrator. You add a new agent to this kit and wire it in so it is actually spawned.

Work directly — do not delegate the writing. You may spawn Explore agents for the analysis in Step 2.

Ask every question with the **AskUserQuestion** tool, 2 to 4 options. Never ask in plain prose — that ends the turn and looks like the skill stopped. The tool always adds a free-text answer of its own, so the user can always type something you did not list.

## Before you start

Unlike every other skill in this kit, you do NOT need `.claude/docs/PROJECT_CONTEXT.md`. You work on agent definitions, not on product code.

Your target is either the plugin repository (a directory with `.claude-plugin/plugin.json` and `agents/`) or a project's `.claude/agents/`. Step 4 decides which. Until then, read only.

---

**1. ASK — What does the new agent do?**
Answers:
- "Describe it:"
- "use an existing agent file"
    - followed by question "where can i find the existing agent?"

Read an imported file now. Keep its role — you rewrite everything around it in Step 6.
If the user passed a description or a path as an argument, skip this step and confirm what you understood.

**2. ANALYZE — Where to locate the new agent?**
Read the existing agents, the skills, the pipeline. Look for all fitting spots and find the best.

Also note the `##` headings already taken in the shared context file, and check honestly whether an existing agent already does this job.

**3. ASK — Where to locate the new agent?**
Answers:
- List of spots, each with a reason, one recommended.
- "somewhere else:"

Add "Extend `<existing agent>` instead" when Step 2 showed the job is already covered. If you concluded the agent should not exist, say so and recommend that option. The user decides; if they overrule you, build it in full without arguing again.

**4. ASK — Who will use the agent?**
Answers:
- "Only this project" (local file, no release)
- "the whole team" (goes into the plugin, needs a version bump)
    - followed by question "You need a copy of the plugin repo. Where is it?"
      Answers:
        - "here:" (path)
        - "create a fork for me" (forks and clones it)
      Never write into the installed plugin — it is overwritten on every update.

Verify the path: it must hold `.claude-plugin/plugin.json` and `agents/`, and the working tree must be clean. This answer decides the memory directory in Step 6.

**5. SHOW — Preview.**
Show every decision and the files that will be edited. then ask.
Answers:
- go
- adjust
- cancel

Decide everything not asked yourself — name, boundary, tools, findings blocking or advisory, memory directory, `##` heading, model — and show all of it here. This is the only place the user sees those decisions. Write nothing before "go".

If the agent needs project facts `PROJECT_CONTEXT.md` does not carry, warn separately: it forces a `CONTRACT_VERSION` bump and a re-adopt in every project.

**6. WRITE — Write the agent file.**
Copy the structure of the existing agents, so it reads the project context, has a memory,
writes to the log, and ends with the phrase the workflow waits for.

Do not copy the memory directory, the `##` heading or the name: the memory prefix follows Step 4's scope, the heading must be unused, and the name must not collide with an agent in this kit, another plugin, or a built-in.

**7. WRITE — Adjust the other files.**
like Skills, adopt, README, version.

For the workflow skills: the step, the spawn prompt (context file path, what to read, what to produce, which heading to append under), the gate the orchestrator checks afterwards, and the final report row. `orchestrate` and `bug-hunt` each have a full and a fast step list. README and version are plugin scope only.

**8. REPORT — What changed, what is still open.**
Only this project: done, the agent works now.
Whole team: user pushes the repo, then everyone runs
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
