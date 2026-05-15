# Time Machine — spec-kit extension

Retroactively apply the full [spec-kit](https://github.com/github/spec-kit) SDD
workflow to an existing codebase. Time Machine analyses your code, groups it
into logical features, and walks each one through the complete lifecycle —
**specify → clarify → plan → tasks → implement** — on its own isolated git
branch, running all features sequentially without stopping.

---

## How it works

```
/speckit.time-machine.analyze   →  scan codebase, build dependency-ordered feature queue
/speckit.time-machine.next      →  run the full SDD loop for ALL pending features in sequence
/speckit.time-machine.status    →  show queue + push status at any time
```

`next` processes every pending feature in one continuous run:

1. Creates `feature/time-machine-<id>` branch for each feature
2. Runs `speckit.specify` → `speckit.clarify` (human Q&A gate) →
   `speckit.plan` → `speckit.tasks` → `speckit.implement`
3. Asks if you're satisfied — redo from any phase, skip, or wrap up and stop
4. Asks whether to push the branch
   - **Yes** → pushes immediately, marks `pushed: true`, continues
   - **No** → moves straight to the next feature without interruption
5. After all features are done, if any branches were not pushed, prints a full
   summary with the exact `git push` commands for each outstanding branch

---

## Installation

```bash
# Install spec-kit (if not already installed)
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# Clone this extension
git clone https://github.com/teeyo/spec-kit-time-machine

# Initialise spec-kit in your existing project
cd /path/to/your-project
specify init --here --integration <your-agent>

# Install the extension
specify extension add --dev /path/to/spec-kit-time-machine
```

> **Note:** The `specify` binary installed by spec-kit lives at
> `~/.local/bin/specify`. If you have the unrelated Specify design-tokens CLI
> on your `PATH`, use the full path `~/.local/bin/specify` instead.

---

## Usage

### 1. Analyse the codebase

```
/speckit.time-machine.analyze
```

Scans your source files, groups them into logical features ordered by
dependency, and writes `.specify/extensions/time-machine/features-queue.yml`.
Pass `--force` to overwrite an existing queue without being prompted.

If no source code is found, the command exits immediately with:

```
There is no existing source code to create specs for.
```

**Example output:**

```
Time Machine queue created — 4 features identified

  #  ID                    Name                     Files
  ─────────────────────────────────────────────────────────
  1  auth                  Authentication            4
  2  user-management       User Management           7
  3  payments              Payments                  5
  4  notifications         Notifications             3

Run /speckit.time-machine.next to begin the first feature.
Run /speckit.time-machine.status at any time to check progress.
```

---

### 2. Run the full workflow

```
/speckit.time-machine.next
```

Processes every pending feature from start to finish. For each feature a header
is printed, all five phases run in sequence, and you choose what to do after
implementation.

**Feature header:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Time Machine — Feature 1 of 4
  ID:    auth
  Name:  Authentication
  Files: src/auth/, src/middleware/auth.js
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Phases run in sequence:**

```
[Phase 1/5] Specify    — "Authentication"
[Phase 2/5] Clarify    — "Authentication"
[Phase 3/5] Plan       — "Authentication"
[Phase 4/5] Tasks      — "Authentication"
[Phase 5/5] Implement  — "Authentication"
```

**Satisfaction gate — after each implementation:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Feature "Authentication" implemented.

Satisfied with the result?
  yes      — continue to the next feature
  redo     — re-run from a specific phase
  skip     — skip this feature and move on
  wrap up  — mark done, commit, and stop for now
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

| Option | Behaviour |
| --- | --- |
| `yes` | Proceeds to the push gate, then starts the next feature |
| `redo` | Asks which phase to restart from; re-runs everything from that point |
| `skip` | Marks the feature `skipped` and immediately moves to the next one |
| `wrap up` | Marks the feature `done`, commits all changes on the branch, and stops — run `next` again to continue from the next feature |

**Push gate:**

```
Push branch "feature/time-machine-auth" to remote? [yes / no]
```

**Final report:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Time Machine complete! 🎉

 All 4 features have been specced and implemented.

   ✓ Pushed:   3
   ↷ Skipped:  1
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### Flags

| Flag | Description |
| --- | --- |
| `--feature <id>` | Start (or resume) from a specific feature |
| `--from <phase>` | Used with `--feature` — resume from a specific phase (`specify`, `clarify`, `plan`, `tasks`, `implement`) |
| `--include-skipped` | Re-run all previously skipped features after any remaining pending ones |
| `--redo <id>` | Force re-run any feature regardless of its current status, resetting it from scratch |

```
# Resume from a specific feature
/speckit.time-machine.next --feature payments

# Resume from a specific phase
/speckit.time-machine.next --feature payments --from plan

# Re-run all skipped features
/speckit.time-machine.next --include-skipped

# Force redo any feature
/speckit.time-machine.next --redo auth
```

**Pause and resume:** If you chose `wrap up`, the queue remembers exactly where
you left off via `current_phase`. Re-run `next` at any time and it picks up
automatically:

```
Resuming 'Payments' from phase: implement
```

---

### 3. Check progress

```
/speckit.time-machine.status
```

Displays a live table of every feature with its status and push state:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Time Machine — Status Report
 Analysed: 2026-05-15T10:00:00Z
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

 Progress: 2/4 features complete

  #   Status        ID                    Name               Pushed  Branch
  ──────────────────────────────────────────────────────────────────────────────
  1   ✓ done        auth                  Authentication     ✓       feature/time-machine-auth
  2   ✓ done        user-management       User Management    ✗       feature/time-machine-user-management
  3   ▶ in-progress payments              Payments           —       feature/time-machine-payments
  4   ○ pending     notifications         Notifications      —       —

 Legend:  ✓ done  ▶ in-progress  ○ pending  ↷ skipped

 Next: Run /speckit.time-machine.next to continue.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Feature queue

Stored at `.specify/extensions/time-machine/features-queue.yml`:

```yaml
analyzed_at: "2026-05-15T10:00:00Z"
project_root: "/path/to/project"
total_features: 4
completed: 2
features:
  - id: "auth"
    name: "Authentication"
    description: "User login, registration, and session management"
    files:
      - src/auth/
    status: done
    current_phase: null
    branch: feature/time-machine-auth
    pushed: true
    started_at: "2026-05-15T10:01:00Z"
    completed_at: "2026-05-15T11:30:00Z"
  - id: "payments"
    name: "Payments"
    files:
      - src/payments/
    status: pending
    current_phase: null
    branch: null
    pushed: null
    started_at: null
    completed_at: null
```

---

## Hooks

| Hook              | Fires after          | Behaviour                                                                                    |
| ----------------- | -------------------- | -------------------------------------------------------------------------------------------- |
| `after_implement` | `/speckit.implement` | Optional prompt to continue to the next feature (useful after choosing `wrap up` mid-queue). |

---

## Configuration (optional)

After installation, copy the config template:

```bash
cp .specify/extensions/time-machine/time-machine-config.template.yml \
   .specify/extensions/time-machine/time-machine-config.yml
```

Available options (branch prefix, ignore patterns, max features) are documented
inside the template file.

---

## Requirements

- `spec-kit >= 0.1.0`
- `git` (for branch creation and pushing)

---

## License

MIT — see [LICENSE](LICENSE)
