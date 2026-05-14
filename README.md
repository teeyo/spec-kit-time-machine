# Time Machine — spec-kit extension

Retroactively apply the full [spec-kit](https://github.com/github/spec-kit) SDD
workflow to an existing codebase. Time Machine analyses your code, groups it
into logical features, and then walks each one through the complete lifecycle —
**specify → clarify → plan → tasks → implement** — on its own isolated git
branch, one feature at a time.

---

## How it works

```
/speckit.time-machine.analyze   →  scans codebase, builds feature queue
/speckit.time-machine.next      →  spec-kit full loop for next feature
/speckit.time-machine.status    →  show queue progress at any time
```

Each call to `next`:

1. Picks the first `pending` feature from the queue
2. Creates `feature/time-machine-<id>` branch
3. Runs `speckit.specify` → `speckit.clarify` (with human Q&A gate) →
   `speckit.plan` → `speckit.tasks` → `speckit.implement`
4. Asks if you're satisfied — or want to redo from any phase
5. Optionally pushes the branch, then moves to the next feature

---

## Installation

```bash
# Install spec-kit if you haven't already
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git

# Clone this extension
git clone https://github.com/te3yo/spec-kit-time-machine

# Install it into your project
cd /path/to/your-project
specify extension add --dev /path/to/spec-kit-time-machine
```

---

## Usage

### 1. Analyse the codebase

```
/speckit.time-machine.analyze
```

Generates `.specify/extensions/time-machine/features-queue.yml` with a
dependency-ordered list of features found in your code.

### 2. Run the first feature

```
/speckit.time-machine.next
```

Walks through the full SDD loop for the next pending feature. You will be
prompted to answer clarification questions mid-flow.

Jump to a specific feature or phase:

```
/speckit.time-machine.next --feature payments
/speckit.time-machine.next --feature payments --from plan
```

### 3. Check progress

```
/speckit.time-machine.status
```

---

## Configuration (optional)

After installation, copy the config template:

```bash
cp .specify/extensions/time-machine/time-machine-config.template.yml \
   .specify/extensions/time-machine/time-machine-config.yml
```

Available options are documented inside the template file.

---

## Requirements

- `spec-kit >= 0.1.0`
- `git` (for branch creation and pushing)

---

## License

MIT — see [LICENSE](LICENSE)
