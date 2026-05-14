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
3. Asks if you're satisfied — or want to redo from any phase
4. Asks whether to push the branch
   - **Yes** → pushes immediately, marks `pushed: true`, continues
   - **No** → moves straight to the next feature without interruption
5. After all features are done, if any branches were not pushed, prints a full
   summary with the exact `git push` commands for each outstanding branch

### Automatic detection on `speckit.constitution`

When you run `/speckit.constitution`, Time Machine automatically checks whether
your project has existing source files:

- **Existing codebase** — the extension fires and asks **once**:
  *"Existing source code detected. Would you like to analyse the codebase and build a feature queue?"*
  Answering yes kicks off `analyze` immediately.
- **New / empty project** — the extension does **nothing**. No prompt, no
  interruption. Time Machine is designed for brownfield projects and stays
  completely out of the way when there is nothing to retroactively spec.

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

Pass `--force` to overwrite an existing queue.

### 2. Run the full workflow

```
/speckit.time-machine.next
```

Processes every pending feature from start to finish without stopping between
them. For each feature you will be asked clarification questions mid-flow and
whether to push the branch after implementation.

Resume from a specific feature or phase:

```
/speckit.time-machine.next --feature payments
/speckit.time-machine.next --feature payments --from plan
```

### 3. Check progress

```
/speckit.time-machine.status
```

Displays a table of every feature with its status (`done`, `in-progress`,
`pending`, `skipped`) and whether its branch was pushed.

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
    branch: feature/time-machine-auth
    pushed: true
    started_at: "2026-05-15T10:01:00Z"
    completed_at: "2026-05-15T11:30:00Z"
  - id: "payments"
    name: "Payments"
    files:
      - src/payments/
    status: pending
    branch: null
    pushed: null
    started_at: null
    completed_at: null
```

---

## Hooks

| Hook                 | Fires after             | Behaviour                                                                                                           |
| -------------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `after_constitution` | `/speckit.constitution` | Checks for existing source files. **Prompts to run `analyze` only if source files are found.** Completely silent and inactive on new empty projects. |
| `after_implement`    | `/speckit.implement`    | Optional prompt to continue to the next feature (useful if you stopped mid-queue).                                  |

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
