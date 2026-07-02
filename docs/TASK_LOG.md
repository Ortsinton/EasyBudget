# EasySaving — Task Log

This document complements the git history: while `git log` tells you *what*
changed, this log captures the *why* behind decisions made in each task, the
problems encountered during implementation, and any follow-up/debt left
pending. Every new Claude Code session should read the most recent entry
before starting the next task on the Trello board.

One entry is added per completed task, in chronological order.

---

## Task 1: Define domain models

**Branch/PR:** `1-models` → merged into `main` in PR #1
**Commits:** `312e78a` (implementation), `367da67` (syntax fix)
**References:** ADR-001, ADR-006, ADR-007 (new, see below)

### Summary

Created the three domain data classes requested (`Transaction`, `Category`,
`Money`) in `shared/src/commonMain/kotlin/com/ortsinton/easysaving/domain/model/`,
with no dependency on Android, iOS or SQLDelight. Added `kotlinx-datetime` as
a multiplatform dependency to represent dates.

### Decisions made (with future implications — see ADR-007)

- **`Long` auto-increment IDs, not UUID.** There's no remote sync in the MVP
  (ADR-006), so the problem UUID solves (collisions between devices) doesn't
  apply yet. Revisit if remote sync is added.
- **`Transaction` references `categoryId: Long`, doesn't embed `Category`.**
  Domain models relationships the way a relational schema would (foreign
  key), avoiding duplicating category data on every transaction. "Enriched"
  views (transaction + resolved category) are the responsibility of
  `presentation`/`data`, not this base model.
- **Dates via `kotlinx-datetime`, not `java.util.Date` or `Foundation.Date`.**
  `Transaction.date` is `LocalDate` (just the expense's date, no time). If
  stable ordering between same-day transactions is ever needed, add a
  separate field (`createdAt: Instant`) instead of mixing it into `date`.
- **`Money` is a `value class` backed by `Long` in cents, not `Double`.**
  Avoids floating-point rounding errors in amounts.
- **`icon` and `color` in `Category` are `String`**, not platform types
  (`Bitmap`, `UIColor`, Compose `Color`). `icon` is a semantic key (e.g.
  `"restaurant"`) and `color` a hex value (`"#FF6B35"`); each platform
  decides how to render it natively.

### Problems encountered during implementation

- Files were initially created under `shared/src/commonMain/domain/`
  (missing the `kotlin/` segment), so Android Studio didn't recognize them
  as Kotlin source and syntax highlighting didn't work. Fix: all Kotlin code
  in a KMP source set must live under `.../<sourceSet>/kotlin/...`.
- Adding `kotlinx-datetime` to the version catalog (`libs.versions.toml`)
  hit two chained errors: (1) the alias was named `androidx-datetime`
  instead of `kotlinx-datetime` (the typesafe accessor is generated from the
  alias key, not from the artifact name), and (2) a typo in the module
  coordinate (`korlinx-datetime` instead of `kotlinx-datetime`). Gradle sync
  didn't fail because the catalog itself was valid — the failure only
  showed up when actually resolving the dependency, and it did so across
  all 7 compilation targets at once (androidMain, androidHostTest,
  iosArm64Main/Test, iosSimulatorArm64Main/Test), which is the expected
  behavior: a `commonMain` dependency propagates to every target
  automatically, there's no need to declare it per target.
- `kotlinx-datetime` version: confirmed `0.8.0` as the latest stable release
  (May 2026) by checking the official repo, instead of assuming the `0.7.1`
  already sitting in cache as a transitive dependency.

### Follow-up generated (not resolved in this task)

- Pending in Trello: **"Clean up Compose Multiplatform scaffolding in
  `:shared`"** — remove `App.kt`, `Greeting.kt`, `GreetingUtil.kt`,
  `Platform.kt` and the Compose dependencies in `commonMain.dependencies`
  of `shared/build.gradle.kts`, since they contradict ADR-001 (100% native
  UI). This task hasn't been executed yet.

### Final state

Code review (`/code-review` skill, medium level) → **Approved**, no
blocking findings.
