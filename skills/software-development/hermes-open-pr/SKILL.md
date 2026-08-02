---
name: hermes-open-pr
description: Create a Hermes Agent PR after the quality gates pass.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes-agent, contribution, pull-request, github]
    related_skills: [hermes-assurance-gate, hermes-pr-feedback-loop, hermes-contributing-check]
---

# Hermes Open PR

Last step of the Hermes Agent contribution lifecycle: open a pull request
against the upstream repository after the local quality gates have passed.
Generates the PR body, pushes the branch to your fork, opens the PR, and posts
a review summary.

## Overview

This skill assumes the change is already committed on a feature branch and the
quality gates (`hermes-assurance-gate`) are GREEN. It does NOT re-run those
checks — it only handles the GitHub-side actions. The fork and remote names are
parameters you supply; nothing here is hard-coded to a specific user or fork.

## When to Use
- After `hermes-assurance-gate` reports GREEN (compile, import side-effect,
  tests, clean tree, commit format, rebase on upstream/main).
- User says "open the PR", "push and create a pull request", "ship it".

Don't use for:
- Opening a PR on a repo other than hermes-agent (use `gh pr create` directly).
- Re-opening or updating an already-merged PR.

## Configuration (set per environment)
These are placeholders — substitute your own values:
- `UPSTREAM` = `NousResearch/hermes-agent` (the canonical repo; PR base).
- `FORK_REMOTE` = your fork's git remote name (e.g. `origin` if you forked, or a
  named remote like `fork`). Default suggestion: `fork`.
- `FORK_OWNER` = your GitHub username (used in `--head <owner>:<branch>`).
- `GH` = `gh` if on PATH; on Windows MSYS use the full path
  `C:\Program Files\GitHub CLI\gh.exe` (it is NOT on the MSYS PATH).

## Step 1 — Write the PR body
Save to `PR_DESCRIPTION.md` (repo root, untracked — do not commit it). Structure:
```
# <Conventional-Commits title, no backticks>

## What changed
<one paragraph: what + why>

## How to test
<exact commands, e.g. `pytest tests/hermes_cli/test_update_concurrent_quarantine.py -v`>

## Platforms tested
Windows / macOS / Linux (be honest; Windows-only changes say so)

## Related
Fixes #<n> / relates to #<n> (if any)

## Notes for reviewers
<anything the maintainer should know: scope limits, known follow-ups>
```
For updates after review, fold the review-reply into this body or post it as a
comment (see `hermes-pr-feedback-loop`).

Completion: `PR_DESCRIPTION.md` exists and contains all six sections.

## Step 2 — Push the branch
```
git push -u <FORK_REMOTE> feature/<slug>        # first time
# or, if the branch is already on the fork:
git push <FORK_REMOTE> feature/<slug>           # fast-forward, no force
```
Only `--force-with-lease` if the local branch diverged from the fork's history
(e.g. after a rebase that rewrote already-pushed commits). ASK before forcing.

Completion: `git status` shows the branch is up to date with `<FORK_REMOTE>/feature/<slug>`.

## Step 3 — Open the PR
```
GH pr create \
  --repo NousResearch/hermes-agent \
  --head <FORK_OWNER>:feature/<slug> \
  --base main \
  --title "<Conventional Commits title>" \
  --body-file PR_DESCRIPTION.md
```
Capture the returned PR URL.

Completion: `GH pr view <N>` returns the open PR with the correct base (`main`)
and head (`<FORK_OWNER>:feature/<slug>`).

## Step 4 — Post review summary (optional but advised)
If the PR addresses prior review (see `hermes-pr-feedback-loop`), post a comment
summarizing which items are closed:
```
GH pr comment <N> --repo NousResearch/hermes-agent --body-file PR_REPLY.md
```

## Common Pitfalls
1. **`gh` not on MSYS PATH (Windows)** — call by full path
   `C:\Program Files\GitHub CLI\gh.exe`, not bare `gh`.
2. **Backticks in `git commit -m` from bash** — command substitution corrupts the
   message; use straight quotes.
3. **Force-push without consent** — only `--force-with-lease` after asking.
4. **Committing PR_DESCRIPTION.md / PR_REPLY.md** — keep them untracked; they are
   not part of the PR diff.
5. **Wrong base** — always `--base main` against `NousResearch/hermes-agent`.

## Verification Checklist
- [ ] `PR_DESCRIPTION.md` has all six sections.
- [ ] Branch pushed to `<FORK_REMOTE>` (fast-forward, no force unless approved).
- [ ] `gh pr view <N>` shows base `main`, head `<FORK_OWNER>:feature/<slug>`.
- [ ] PR URL captured and reported to the user.
- [ ] No stray `*.md` / `*.patch` files staged in the PR.
