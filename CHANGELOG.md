# Changelog

## [1.1.0] - 2026-05-15

### Added

- `--include-skipped` flag on `next` — re-runs all previously skipped features after any remaining pending ones
- `--redo <id>` flag on `next` — force re-runs any feature regardless of its current status, resetting it from scratch
- `wrap up` satisfaction option — marks a feature done, commits the branch, and exits the queue gracefully
- `current_phase` field in `features-queue.yml` — tracks the active phase so resuming after `wrap up` automatically continues from the correct phase without any flags

### Changed

- `analyze` now checks for existing source code upfront and exits with a clear message if none is found
- Satisfaction gate option renamed from `pause` to `wrap up` with commit-based behaviour
- `next` resumes `in-progress` features from the last recorded phase automatically

### Removed

- `speckit.time-machine.bootstrap` command — source code detection is now handled directly by `analyze`
- `after_constitution` hook — no longer needed

---

## [1.0.0] - 2026-05-14

### Added

- `speckit.time-machine.analyze` — scan codebase and build dependency-ordered feature queue
- `speckit.time-machine.next` — full SDD loop (specify → clarify → plan → tasks → implement) per feature with git branch isolation, satisfaction gate, and push gate
- `speckit.time-machine.status` — progress table for the feature queue
- `after_implement` hook (optional) — prompts to continue to the next feature automatically
- Config template for branch prefix and skip patterns
