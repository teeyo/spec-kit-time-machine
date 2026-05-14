# Changelog

## [1.0.0] - 2026-05-14

### Added

- `speckit.time-machine.analyze` — scan codebase and build dependency-ordered feature queue
- `speckit.time-machine.next` — full SDD loop (specify → clarify → plan → tasks → implement) per feature with git branch isolation, satisfaction gate, and push gate
- `speckit.time-machine.status` — progress table for the feature queue
- `after_implement` hook (optional) — prompts to continue to the next feature automatically
- Config template for branch prefix and skip patterns
