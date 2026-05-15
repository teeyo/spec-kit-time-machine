---
description: "Scan the codebase and generate a sequenced feature queue for the Time Machine workflow"
---

# Time Machine — Analyze

Scan the current project and produce a feature queue file so `speckit.time-machine.next`
can process each feature through the full SDD lifecycle.

## User Input

$ARGUMENTS

Accepted flags:

- `--force` — overwrite an existing `features-queue.yml` without asking

---

## Steps

### 1. Check for existing source code

Scan the project root for source files, excluding:

- `.specify/`
- `.github/`
- `node_modules/`
- `dist/`, `build/`, `out/`
- `.git/`
- `vendor/`, `__pycache__/`, `.venv/`
- Common lock files (`*.lock`, `package-lock.json`, `yarn.lock`)

If **no source files are found**, stop and tell the user:

```
There is no existing source code to create specs for.
```

If source files are found, continue.

---

### 3. Check for an existing queue

Look for `.specify/extensions/time-machine/features-queue.yml`.

- If it exists **and** `--force` was **not** passed:
  - Ask the user: "A queue already exists. Overwrite it? [yes / no]"
  - If the answer is "no", stop and tell the user to run
    `/speckit.time-machine.status` to see the current queue.

---

### 4. Understand the project

Read the following to build context before identifying features:

- Top-level directory listing
- `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, or
  equivalent dependency manifest (whichever exists)
- Any existing `README.md` at the project root
- Any files inside `.specify/` (existing specs, constitution, etc.)

Ignore these paths entirely — they are not source features:
`node_modules/`, `dist/`, `build/`, `.git/`, `vendor/`, `__pycache__/`,
`.venv/`, `coverage/`, `*.lock`, `*.log`

---

### 5. Identify features

Group the remaining source files into **logical, user-facing features**. Each
feature should represent a cohesive unit that can be specced and implemented
independently (e.g. "Authentication", "Payments", "Notifications", "User Profile").

Rules:

- Target **3–15 features**. Merge very small subsystems; split very large ones.
- Assign each feature a short, URL-safe **id**: lowercase letters and hyphens
  only (e.g. `auth`, `user-management`, `email-notifications`).
- **Order by dependency** — foundational features first (e.g. `auth` before
  features that call `requireAuth()`).
- List the key files/directories relevant to each feature (relative paths from
  project root).

---

### 6. Write the queue file

Create `.specify/extensions/time-machine/` if it does not already exist.

Write `.specify/extensions/time-machine/features-queue.yml` using **exactly**
this structure:

```yaml
analyzed_at: "<ISO 8601 timestamp, e.g. 2026-05-14T10:00:00Z>"
project_root: "<absolute path to project root>"
total_features: <integer>
completed: 0
features:
  - id: "<feature-id>"
    name: "<Human-Readable Feature Name>"
    description: "<One sentence describing what this feature does for users>"
    files:
      - "<relative/path/to/file-or-directory>"
    status: pending # pending | in-progress | done | skipped
    current_phase: null # null | specify | clarify | plan | tasks | implement
    branch: null    pushed: null           # null | true | false    started_at: null
    completed_at: null
  # ... repeat for each feature
```

---

### 7. Confirm to the user

Print a summary in this format:

```
Time Machine queue created — <N> features identified

  #  ID                    Name                     Files
  ─────────────────────────────────────────────────────────
  1  auth                  Authentication            4
  2  user-management       User Management           7
  3  payments              Payments                  5
  ...

Run /speckit.time-machine.next to begin the first feature.
Run /speckit.time-machine.status at any time to check progress.
```
