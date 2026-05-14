---
description: "Run the full SDD workflow for every pending feature in the Time Machine queue, sequentially"
---

# Time Machine — Next Feature

Process every pending feature in the queue **one after the other** without
stopping between them. For each feature the full spec-kit SDD lifecycle runs:
**specify → clarify → plan → tasks → implement**. After each implementation
the user is asked whether to push the branch. Declining does **not** stop the
loop — the next feature starts immediately. A final summary is shown when all
features are done.

## User Input

$ARGUMENTS

Accepted flags:

- `--feature <id>` — start (or resume) from a specific feature instead of the next pending one
- `--from <phase>` — when used with `--feature`, resume that feature from a specific phase (`specify`, `clarify`, `plan`, `tasks`, `implement`)

---

## Steps

> **This command runs a loop.** Repeat Steps 1–10 for every remaining
> `pending` feature in order. Do not stop between features. Only exit the
> loop when all features are `done` or `skipped`, then proceed to Step 11.

---

### 1. Load and validate the queue

Read `.specify/extensions/time-machine/features-queue.yml`.

- If the file does not exist → stop and tell the user:
  `"No queue found. Run /speckit.time-machine.analyze first."`
- Collect every feature where `status` is `pending` or `in-progress`, in order.
- If `--feature <id>` was passed → start the loop from that feature only
  (skip any pending features before it).
- If no pending or in-progress features remain → skip straight to Step 11.

---

### 2. Mark as in-progress

For the current feature, update `features-queue.yml`:

```yaml
status: in-progress
started_at: "<current ISO 8601 timestamp>" # only if not already set
```

Print:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Time Machine — Feature <#N of total>
  ID:    <feature-id>
  Name:  <Feature Name>
  Files: <comma-separated list>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### 3. Create a git branch

```bash
git checkout -b feature/time-machine-<feature-id>
```

- If the branch already exists (resuming), run `git checkout feature/time-machine-<feature-id>` instead.
- Update `features-queue.yml` → `branch: "feature/time-machine-<feature-id>"`.
- If git is not initialised, warn the user and continue without branching.

---

### 4. Phase: Specify

```
[Phase 1/5] Specify — "<Feature Name>"
```

Run `/speckit.specify` scoped to this feature, passing:

> "Feature: <Feature Name>. Description: <feature description>.
> Relevant files: <comma-separated file list>.
> Focus on this feature only; do not modify other features."

Wait for the spec to be written before continuing.
If `--from` is set to a later phase, skip this step.

---

### 5. Phase: Clarify

```
[Phase 2/5] Clarify — "<Feature Name>"
```

Run `/speckit.clarify` for this feature.

**Human gate:** present every generated question to the user and wait for
answers before proceeding. Do not auto-answer or skip.

If `--from` is set to a later phase, skip this step.

---

### 6. Phase: Plan

```
[Phase 3/5] Plan — "<Feature Name>"
```

Run `/speckit.plan` for this feature. Wait for the plan to be written.
If `--from` is set to a later phase, skip this step.

---

### 7. Phase: Tasks

```
[Phase 4/5] Tasks — "<Feature Name>"
```

Run `/speckit.tasks` for this feature. Wait for tasks to be generated.
If `--from` is set to a later phase, skip this step.

---

### 8. Phase: Implement

```
[Phase 5/5] Implement — "<Feature Name>"
```

Run `/speckit.implement` for this feature. Wait for implementation to complete.

---

### 9. Satisfaction gate

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Feature "<Feature Name>" implemented.

Satisfied with the result?
  yes    — continue
  redo   — re-run from a specific phase
  skip   — skip this feature and move on
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- **yes** → proceed to Step 10.
- **redo** → ask `"Which phase? [specify / clarify / plan / tasks / implement]"`
  then loop back to the relevant step above.
- **skip** → update `status: skipped`, `completed_at: <now>`, `pushed: null`.
  Print a one-line skip notice then **continue immediately to the next feature**
  (go back to Step 2 with the next pending feature).

---

### 10. Push gate

Ask the user:

```
Push branch "feature/time-machine-<feature-id>" to remote? [yes / no]
```

**If yes:**

- Run `git push -u origin feature/time-machine-<feature-id>`.
- Update `features-queue.yml`: `pushed: true`.
- Print a one-line confirmation.

**If no:**

- Update `features-queue.yml`: `pushed: false`.
- Print nothing extra — **immediately continue to the next feature**.

Either way, update the feature:

```yaml
status: done
completed_at: "<current ISO 8601 timestamp>"
```

Then go back to Step 2 with the next pending feature.

---

### 11. All features done — final report

Reached when every feature is `done` or `skipped`.

Collect all features where `pushed: false` (declined) and `status: done`.

**If every done feature was pushed (`pushed: true` or `pushed: null` for skipped):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Time Machine complete! 🎉

 All <N> features have been specced and implemented.

   ✓ Pushed:   <N>
   ↷ Skipped:  <N>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**If any done features were NOT pushed (`pushed: false`):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Time Machine complete!

 Here's a summary of all branches created:

  Branch                                  Status
  ──────────────────────────────────────────────────────
  feature/time-machine-<id-1>             ✓ pushed
  feature/time-machine-<id-2>             ✗ not pushed
  feature/time-machine-<id-3>             ✗ not pushed
  feature/time-machine-<id-4>             ↷ skipped
  ...

 You chose to push <N-pushed> of <N-done> branches.

 Don't forget to push the remaining branches when you're
 ready — since you preferred to handle that yourself,
 here are the commands:

<for each pushed: false feature, one line:>
   git push -u origin feature/time-machine-<id>

 Take your time — Spec Kit Time Machine has done its part. 🚀
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
