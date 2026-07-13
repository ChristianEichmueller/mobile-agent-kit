<!-- mobile-kit context contract — CONTRACT_VERSION: 1 -->
<!--
This file is the entry point every mobile-kit agent reads before doing anything.
Copy it to `.claude/docs/PROJECT_CONTEXT.md` in your project and fill it in,
or run `/mobile-kit:adopt` to generate it automatically from your codebase.

Rules for filling it in:
- Keep this file SHORT. Link out to detailed docs instead of inlining them.
- Every section marked REQUIRED must be filled — agents will refuse guesswork.
- Sections marked OPTIONAL may be deleted if not applicable.
-->

# Project Context

CONTRACT_VERSION: 1

## Identity (REQUIRED)

- **Project name:** <name>
- **Stack:** <one of: kmm | flutter | android | other — with a one-line description, e.g. "Kotlin Multiplatform Mobile: shared business logic, native Android (Views/Compose) and iOS (Swift/UIKit) UIs">
- **Modules / top-level layout:** <e.g. `shared/`, `androidApp/`, `iosApp/` — one line each>

## Reference documents (REQUIRED)

Agents must read these before writing code. List every document that defines
architecture, patterns, and conventions, with a one-line description:

- `<path>` — <architectural patterns, repository/service guidelines>
- `<path>` — <codebase structure overview>
- `<path>` — <migration guides, refactoring trackers — OPTIONAL>

## Build commands (REQUIRED)

```bash
# Build the app (the command agents use to verify compilation)
<command>

# Build showing ALL errors at once (if different)
<command>
```

<Any build rules, e.g. "always build the app module, never the shared module directly">

## Test commands and policy (REQUIRED)

```bash
# Run the full test suite agents should use as the green gate
<command>

# Run a single test class
<command>

# Run a single test method
<command>
```

- **Test policy:** <what kinds of tests this project uses and FORBIDS,
  e.g. "UI/instrumented tests only, never unit tests" or "widget tests + integration tests">
- **Test location(s):** `<path>`
- **Test file organization:** <e.g. "one file per screen/flow; add to existing files, new files are the exception">
- **Emulator/device policy:** <e.g. "always emulators/simulators, never real devices unless explicitly asked">

## Test infrastructure (REQUIRED)

The reusable helpers agents MUST use instead of re-implementing:

- **Fakes / test doubles:** <e.g. class names + where they live>
- **Scenario / fixture setup:** <the standard pattern for seeding state>
- **UI-test abstractions:** <robot classes / page objects / finders + location>
- **Base test classes:** <lifecycle, permissions, teardown helpers>

## Source layout and naming (REQUIRED)

- **Package/namespace root:** `<e.g. com.example.app>`
- **Feature structure:** <e.g. "features/<feature>/data|domain|mapper" or "lib/features/<feature>/...">
- **Naming conventions:** <e.g. `{Feature}Repository`, `{Feature}ViewModel`, `{Feature}Fragment` / `{Feature}Screen`>
- **Key base classes / patterns to extend:** <e.g. repository base classes, bloc/cubit conventions>

## Code style (REQUIRED)

<Project-specific style rules agents must obey, e.g.:>
- <comment policy>
- <documentation policy>
- <import rules>

## Commit conventions (REQUIRED)

- **Format:** <e.g. `Scope: Component: brief description`>
- **Rules:** <length limits, single/multi line, forbidden content>
- Agents never commit — the user reviews and commits manually.

## Design system (OPTIONAL — required if the project has UI work)

- **Theme / design tokens:** <where colors, typography, spacing, shapes are defined>
- **Shared UI components:** <where reusable components live>
- **Hard rules:** <e.g. "no hardcoded colors/dimensions/strings in layouts; always resources/tokens">

## Project quirks (OPTIONAL)

Anything that would surprise a competent developer new to this codebase:

- <e.g. "module X is frozen until <date> — do not touch">
- <e.g. "screen Y has a known flaky navigation path in tests — avoid for regression tests">
