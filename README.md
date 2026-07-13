# mobile-agent-kit

A [Claude Code plugin](https://code.claude.com/docs/en/plugins) providing a TDD-style multi-agent orchestration suite for mobile projects (Android/KMM and Flutter). One shared, versioned agent setup — adapted to each project through a generated context pack, with per-project agent memory.

## What's inside

**9 agents** (`agents/`):

| Agent | Role |
|---|---|
| `mobile-planner` | Requirements discovery + implementation plans with test plan and phased order |
| `mobile-developer` | Implements plans precisely; makes failing tests green; documents deviations |
| `test-writer` | Writes failing UI/integration tests BEFORE implementation; defends or adjusts them on push-back |
| `bug-fixer` | Root-cause diagnosis first, then the smallest correct fix; never touches tests |
| `tech-lead` | Architecture/SOLID/convention review; also hunts unnecessary indirection |
| `qa-reviewer` | Paranoid edge-case and bug review; assesses test coverage gaps |
| `code-optimizer` | Finds dead abstractions, redundant layers, data-flow detours |
| `design-analyzer` | Design requirements + reusable-component mapping |
| `design-system-guardian` | Design-token compliance, no hardcoded values, accessibility |

**5 skills** (`skills/`):

| Command | Workflow |
|---|---|
| `/mobile-kit:adopt` | Onboard a project: analyze codebase, generate the context pack, scaffold memory |
| `/mobile-kit:orchestrate <task>` | Full feature workflow: plan → failing tests → implement → parallel review → design review |
| `/mobile-kit:bug-hunt <bug>` | Diagnose → failing repro test → fix → review |
| `/mobile-kit:review-loop` | tech-lead + qa-reviewer + code-optimizer in parallel, fix, re-review until clean |
| `/mobile-kit:implement-plan <plan-path>` | Phase-by-phase implementation with per-phase review |

Key rule across all workflows: **tests are written before implementation and are the contract.** Implementers may not modify tests; disagreements go through a capped push-back protocol (max 2 iterations, then escalate to the user).

## Install

```
/plugin marketplace add ChristianEichmueller/mobile-agent-kit
/plugin install mobile-kit@mobile-agent-kit
/mobile-kit:adopt
```

`adopt` detects your stack (KMM / Flutter / Android), analyzes the codebase, and generates `.claude/docs/PROJECT_CONTEXT.md` — the context pack all agents read at runtime. Review and commit the generated files.

## How adaptation works (3 layers)

1. **Shared (this plugin):** agent roles, TDD discipline, review workflows. Identical everywhere, updated centrally.
2. **Per project (generated, committed):** `.claude/docs/PROJECT_CONTEXT.md` + reference docs — build/test commands, test infrastructure, architecture patterns, conventions. Agents refuse to guess anything listed there.
3. **Per project (runtime):** every agent has `memory: project` — institutional knowledge accumulates in `.claude/agent-memory/mobile-kit-<agent>/` inside each project and never leaves it. (Plugin agents get a plugin-prefixed memory directory; a project-local override agent named `tech-lead` would use plain `.claude/agent-memory/tech-lead/` instead.)

## Updating

Improve agents/skills here, push, then on each consuming machine:

```
/plugin marketplace update mobile-agent-kit   # refresh the catalog
claude plugin update mobile-kit               # or update the plugin from the CLI
```

Or set `"autoUpdate": true` on the marketplace entry in `.claude/settings.json`. After larger updates, re-run `/mobile-kit:adopt` in each project to refresh the context pack (it preserves hand-written content).

## Overriding per project

Project-level definitions always win. To customize one agent for one project, copy it to `<project>/.claude/agents/<name>.md` and edit — the plugin version is ignored for that project while the rest stay shared.

## Migrating from a local-agent setup

If a project previously carried these agents as local `.claude/agents/*.md` files with accumulated memory, the memory directories must be renamed to the plugin-prefixed form before removing the local definitions:

```
git mv .claude/agent-memory/<agent> .claude/agent-memory/mobile-kit-<agent>
```

(For the renamed agents, also change the base name: `android-developer` → `mobile-kit-mobile-developer`, `android-planner` → `mobile-kit-mobile-planner`.)

## Known issues

Claude Code (observed on 2.1.195) injects a broken Persistent Agent Memory path for plugin agents — a doubled `.claude/.claude` segment — so automatic MEMORY.md loading silently fails. The agents in this kit work around it by self-managing memory: they read/write `.claude/agent-memory/mobile-kit-<agent>/` explicitly and ignore the injected path. No action needed; this note can be dropped once the upstream bug is fixed.

## License

MIT
