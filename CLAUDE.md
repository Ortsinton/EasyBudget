# EasySaving — Project Context for Claude Code

This file is read automatically at the start of every Claude Code session.
It contains the minimum context needed to work on any project task without
having to re-explain the architecture in every chat.

## What EasySaving is

Personal finance (budgeting) app built as a portfolio project. Lets users
record expenses/income and view analytics (by category, by date, trends).
KMP project with native Android (Compose) and iOS (SwiftUI), sharing
domain, data and presentation.

Full documentation of decisions and structure:
- `docs/ADR.md` — architecture decisions with context and trade-offs
- `docs/PROJECT_STRUCTURE.md` — module structure and conventions
- `docs/TASK_LOG.md` — history of completed tasks: what was decided, what
  problems came up, and what follow-up was left pending on each one. Read
  the most recent entry before starting a new task.

## Tech stack

- Kotlin Multiplatform (targets: android, iosArm64, iosSimulatorArm64)
- Local persistence: SQLDelight (offline-first, no backend)
- DI: Koin
- Async: Coroutines + Flow
- Swift interop: SKIE
- UI: Jetpack Compose (Android) and modern SwiftUI with @Observable (iOS 17+)
- Shared testing: kotlin.test + Turbine
- CI: GitHub Actions

## Sharing boundary (most important rule)

- **Shared:** domain (models, use cases), data (repositories, SQLDelight),
  presentation (ViewModels)
- **Native:** full UI (Compose / SwiftUI), navigation, charts

Shared ViewModels NEVER navigate. They expose:
- A `StateFlow<UiState>` with the state to render
- Action functions (e.g. `onTransactionSelected(id)`) that only notify
  intent; each platform decides whether that means navigating and how

See ADR-003 and ADR-004 for the full detail and the reasoning behind this
decision.

## Code conventions

- Packages: `com.ortsinton.easysaving.domain`, `com.ortsinton.easysaving.data`,
  `com.ortsinton.easysaving.presentation`, `com.ortsinton.easysaving.di`
- `shared/domain` must not import anything from Android, iOS or SQLDelight
- `shared/presentation` must not import SQLDelight types directly (always
  go through domain)
- All business logic (calculations, validation, aggregations) lives in
  domain, never in a ViewModel or a Composable/View
- Domain entity modeling conventions (`Long` auto-increment IDs, no UUID;
  relationships by id, not embedded objects; dates with `kotlinx-datetime`;
  amounts as a `value class` over `Long` in cents; icon/color as `String`):
  see ADR-007 before creating a new entity
- Documentation, code comments, commit messages and PR descriptions are
  written in English — this repo is meant to be read by contributors
  worldwide. Conversation between the user and Claude Code stays in
  whichever language the user uses in chat (normally Spanish); this rule
  only applies to what gets committed to the repo.

## Task workflow

1. Each task comes from a Trello card (EasySaving board, "To Do" list) with
   Objective + Acceptance criteria + ADR references.
2. Create a new branch per task: `task-N-short-name`
3. When done, verify the project builds for both targets and tests pass
4. Descriptive commit referencing the task, e.g.:
   `feat: define domain models (Transaction, Category, Money)`
5. Progress between sessions is inferred from the code and git history, not
   from previous conversations — every Claude Code session starts with no
   memory of previous sessions, which is why this file, the code and
   `docs/TASK_LOG.md` are the single source of truth. When a task is
   finished, add a new entry to `docs/TASK_LOG.md` following the same
   format as the previous ones.

## What NOT to do

- No sharing UI across platforms (no Compose Multiplatform): the project's
  goal is to demonstrate native mastery of both toolkits
- No navigation inside a shared ViewModel
- No features outside the MVP (budgets, multi-account, multi-currency,
  remote sync) unless explicitly requested — they're documented as future
  roadmap in the ADR
