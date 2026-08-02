---
name: hermes-assurance-gate
description: Run the software-assurance gate before a Hermes Agent PR.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes-agent, contribution, quality-gate, pre-pr]
    related_skills: [hermes-contributing-check, hermes-open-pr, hermes-pr-feedback-loop]
---

# Hermes Assurance Gate

Final go/no-go quality gate before opening a Hermes Agent PR. Runs concrete
checks on the working tree and blocks the PR if any step fails.

## Overview

This skill is the local quality barrier in the Hermes contribution lifecycle.
It assumes a change is committed on a feature branch and runs compile, import
side-effect, test, and tree-cleanliness checks. It stops at a clean push — PR
creation is delegated to `hermes-open-pr`. Fork/remote names are parameters,
not hard-coded.

## When to Use
- After implementing a Hermes Agent change and before opening a PR.
- User says "run the gate", "check before PR", "quality gate".

Don't use for:
- Non-Hermes repos (adapt the steps manually).
- Doc-only changes where tests/compile are irrelevant (skip, but still check secrets).

## Configuration (set per environment)
- `UPSTREAM` = `NousResearch/hermes-agent` (PR base).
- `FORK_REMOTE` = your fork's git remote name (e.g. `fork`).
- `GH` = `gh` (full path on Windows MSYS: `C:\Program Files\GitHub CLI\gh.exe`).

## Gate steps (all must pass)
1. **Compile** — `python -m py_compile` on every touched `.py`. Must pass.
2. **Import side-effect** — `python -c "import <module>"` on any module imported
   at startup (e.g. `hermes_cli.update_cmd`). Must produce NO prompt/output and
   must not touch the filesystem. (Catches module-scope code that fires on every
   Hermes launch — see `hermes-agent-contrib` hard rule #1.)
3. **Tests** — `pytest tests/ -q` (or `scripts/run_tests_parallel.py` for GitHub
   Actions parity). Green vs baseline; only NEW failures block.
4. **Clean tree** — `git status --short`: only intended files staged; no
   `*.patch`, `*.backup`, `*.orig`; no secrets (API keys / tokens) in diff.
5. **Commit format** — Conventional Commits `<type>(<scope>): <desc>`; no backticks
   (bash command-substitution corrupts the message).
6. **Rebase** — onto `UPSTREAM/main` (fast-forward push OK; only
   `--force-with-lease` if the branch already had commits — ASK before forcing).
7. **Push** — `git push -u <FORK_REMOTE> feature/<slug>` (or
   `git push <FORK_REMOTE> feature/<slug>` if already on the fork). Fast-forward
   only; ASK before `--force-with-lease`. PR creation is handled by `hermes-open-pr`.

## NO-GO
If any of steps 1-4 fails, STOP and report. Do not push. Fix, then re-run.
Security concerns from `requesting-code-review` also block.

## Common Pitfalls
1. **`gh` not on MSYS PATH (Windows)** — call by full path
   `C:\Program Files\GitHub CLI\gh.exe`.
2. **Backticks in `git commit -m` from bash** — command substitution corrupts the
   message; use straight quotes.
3. **CRLF whole-file diff from `cp`** — use `git show UPSTREAM/main:path` to reset
   with LF only.
4. **Force-push without consent** — ask before `--force-with-lease`.

## Verification Checklist
- [ ] `py_compile` passes on all touched `.py`.
- [ ] `import <module>` produces no side effect (no prompt, no FS write).
- [ ] `pytest` green vs baseline (only NEW failures block).
- [ ] `git status --short` shows only intended files; no secrets / stray patches.
- [ ] Commit message follows Conventional Commits + scope; no backticks.
- [ ] Branch rebased on `UPSTREAM/main`; push is fast-forward (or force approved).
