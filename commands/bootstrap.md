---
description: "Silently check whether this project has existing source code and, if so, offer to start Time Machine"
---

# Time Machine — Bootstrap

This command is invoked automatically via the `after_constitution` hook.
It runs silently and **does nothing** if the project has no existing source code.

## Steps

### 1. Scan for source files

Count source files in the project root, excluding:

- `.specify/`
- `.github/`
- `node_modules/`
- `dist/`, `build/`, `out/`
- `.git/`
- `vendor/`, `__pycache__/`, `.venv/`
- Common lock files (`*.lock`, `package-lock.json`, `yarn.lock`)

### 2. Decision

**If no source files are found** (only spec-kit scaffolding exists):

- Exit immediately. Print nothing. Do not prompt the user.

**If source files are found:**

- Ask the user **once**:

```
Time Machine: Existing source code detected.
Would you like to analyse the codebase and build a feature queue for retroactive spec-driven development?
[yes / no]
```

- If **yes** → run `/speckit.time-machine.analyze`
- If **no** → exit silently. The user can always run `/speckit.time-machine.analyze` manually later.
