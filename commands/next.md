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
- `--include-skipped` — re-run all previously skipped features after any remaining pending ones
- `--redo <id>` — force re-run a specific feature regardless of its current status (`pending`, `in-progress`, `done`, or `skipped`)

---

## Steps

> **This command runs a loop.** Repeat Steps 1–10 for every remaining
> `pending` feature in order. Do not stop between features. Only exit the
> loop when all eligible features are `done` or `skipped`, then proceed to Step 11.
> When `--include-skipped` is passed, `skipped` features are included in the run.
> When `--redo <id>` is passed, only that single feature is processed regardless of status.

---

### 1. Load and validate the queue

Read `.specify/extensions/time-machine/features-queue.yml`.

- If the file does not exist → stop and tell the user:
  `"No queue found. Run /speckit.time-machine.analyze first."`
- **If `--redo <id>` was passed:**
  - Find the feature with that id regardless of its current status.
  - If not found → stop: `"No feature with id '<id>' found in the queue."`
  - Reset it: `status: pending`, clear `completed_at`, `branch`, `pushed`, `started_at`.
  - The loop runs for **only this one feature**.
- **Otherwise:**
  - Collect every feature where `status` is `pending` or `in-progress`, in order.
  - For any collected feature where `status` is `in-progress` and `current_phase` is set,
    automatically treat it as if `--from <current_phase>` was passed for that feature.
    Print: `"Resuming '<Feature Name>' from phase: <current_phase>"`
  - If `--include-skipped` was also passed, append every `skipped` feature (in original order) and reset each to `status: pending`.
  - If `--feature <id>` was passed → start the loop from that feature only
    (skip any features before it in the collected list).
  - If no eligible features remain → skip straight to Step 11.

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

Update `features-queue.yml` → `current_phase: specify`.

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

Update `features-queue.yml` → `current_phase: clarify`.

Run `/speckit.clarify` for this feature.

**Human gate:** present every generated question to the user and wait for
answers before proceeding. Do not auto-answer or skip.

If `--from` is set to a later phase, skip this step.

---

### 6. Phase: Plan

```
[Phase 3/5] Plan — "<Feature Name>"
```

Update `features-queue.yml` → `current_phase: plan`.

Run `/speckit.plan` for this feature. Wait for the plan to be written.
If `--from` is set to a later phase, skip this step.

---

### 7. Phase: Tasks

```
[Phase 4/5] Tasks — "<Feature Name>"
```

Update `features-queue.yml` → `current_phase: tasks`.

Run `/speckit.tasks` for this feature. Wait for tasks to be generated.
If `--from` is set to a later phase, skip this step.

---

### 8. Phase: Implement

```
[Phase 5/5] Implement — "<Feature Name>"
```

Update `features-queue.yml` → `current_phase: implement`.

Run `/speckit.implement` for this feature. Wait for implementation to complete.

---

### 9. Satisfaction gate

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Feature "<Feature Name>" implemented.

Satisfied with the result?
  yes      — continue to the next feature
  redo     — re-run from a specific phase
  skip     — skip this feature and move on
  wrap up  — mark done, commit, and stop for now
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

- **yes** → proceed to Step 10.
- **redo** → ask `"Which phase? [specify / clarify / plan / tasks / implement]"`
  then loop back to the relevant step above.
- **skip** → update `status: skipped`, `completed_at: <now>`, `pushed: null`, `current_phase: null`.
  Print a one-line skip notice then **continue immediately to the next feature**
  (go back to Step 2 with the next pending feature).
- **wrap up** → mark the feature complete, commit all changes on the current branch, then stop:
  1. Run `git add -A && git commit -m "feat(time-machine): complete <feature-id>"`.
  2. Update `features-queue.yml`: `status: done`, `completed_at: <now>`, `current_phase: null`, `pushed: false`.
  3. Print:
     ```
     Feature "<Feature Name>" marked complete and committed.
     Run /speckit.time-machine.next to continue with the next feature.
     ```
  4. Exit the loop. Do **not** proceed to Step 10 or 11.

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
current_phase: null
```

Then go back to Step 2 with the next pending feature.

---

### 11. All features done — final report

Reached when every eligible feature is `done` or `skipped`.

- If `--redo <id>` was used, "eligible" means just that one feature.
- If `--include-skipped` was used, previously skipped features that were re-run now count toward `done`/`pushed`.

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
