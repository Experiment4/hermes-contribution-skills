---
name: hermes-pr-feedback-loop
description: Re-validate Hermes PR review feedback against the gates.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes-agent, contribution, code-review, pull-request]
    related_skills: [hermes-contributing-check, hermes-assurance-gate, hermes-open-pr]
---

# Hermes PR Feedback Loop

Closes the contribution lifecycle: after a PR is published and the automated
reviewer (Sweeper bot) and maintainers (e.g. teknium1) have reviewed it, pull the
feedback in, classify it, and re-validate the change against the SAME gates used
before merge (`hermes-contributing-check` + `hermes-assurance-gate`). Produces a
clear "addressed / not addressed / needs work" verdict per item, and can apply
fixes and re-open the loop.

## Overview

This skill is environment-agnostic: fork/remote names are parameters, not
hard-coded. It assumes a PR already exists against `NousResearch/hermes-agent`
and has received review comments, bot labels, or CI failures. It does not create
the PR (that is `hermes-open-pr`); it processes what came back and drives the
fix→re-check→push cycle.

## When to Use
- User says "address the review", "Sweeper flagged my PR", "teknium commented",
  "re-check PR #NNNN", or after a PR comes back from review.
- Trigger automatically after `hermes-open-pr` if the user wants a self-check loop.

Don't use for:
- Opening a brand-new PR (use `hermes-open-pr`).
- Reviewing someone else's PR without their request (use `gh pr view` read-only).

## Configuration (set per environment)
- `UPSTREAM` = `NousResearch/hermes-agent` (PR base repo).
- `FORK_REMOTE` = your fork's git remote name (e.g. `fork`).
- `FORK_OWNER` = your GitHub username.
- `GH` = `gh` (full path on Windows MSYS: `C:\Program Files\GitHub CLI\gh.exe`).

## Step 1 — Pull the feedback
```
GH pr view <N> --repo NousResearch/hermes-agent \
  --json number,title,state,labels,reviews,comments,body,url
GH api repos/NousResearch/hermes-agent/pulls/<N>/comments \
  --jq '.[] | {user: .user.login, path: .path, line: .line, body: .body}'
GH pr checks <N> --repo NousResearch/hermes-agent
```
Classify each item:
- **Sweeper bot** — usually labels (`sweeper:*`) or a bot comment; auto-generated,
  often a risk/blast-radius tag. Treat labels as signals, not hard blockers.
- **Maintainer (e.g. teknium1)** — inline code comments + review summary. These are
  the real blockers. Quote the exact concern.
- **CI** — failed checks from `gh pr checks`; map to test/lint gates.

Completion: every review item is in one of the three buckets above with its
source and verbatim text captured.

## Step 2 — Re-validate each item against the gates
For every distinct concern, run the matching check from `hermes-contributing-check`:
- "scope too broad" / "mixed refactor" → contributing scope check.
- "breaks cache stability" / "wrong layer" → architecture prompt-cache + footprint check.
- "no tests" / "secret in diff" / "shell injection" → best-practice + assurance gate.
- "import side effect" / "doesn't compile" → assurance gate steps 1-2.
State explicitly: ADDRESSED (change already satisfies it) / NOT ADDRESSED (what's
missing) / N/A.

Completion: every item has a verdict (ADDRESSED / NOT ADDRESSED / N/A) with the
gate it was checked against.

## Step 3 — Apply fixes (only if user approves)
For each NOT ADDRESSED item:
- Propose the concrete edit (file + change).
- If user says go: apply, then re-run `hermes-assurance-gate` (compile, import
  side-effect, tests, clean tree) on the patched tree.
- Re-run Step 2 to confirm closure. Max 2 fix cycles; then report remaining items.

Completion: every approved fix is applied AND re-validated green, or the loop
stopped after 2 cycles with a remaining-items report.

## Step 4 — Push update (if fixes applied)
- `git commit` (Conventional Commits + scope, no backticks).
- Fast-forward push to `<FORK_REMOTE>` (branch already exists → fast-forward OK;
  only `--force-with-lease` if history diverged — ASK first).
- The PR updates automatically; post a reply summarizing which review items are
  now closed (mirrors `hermes-open-pr` Step 4).

Completion: `git status` shows the branch up to date with
`<FORK_REMOTE>/feature/<slug>` and the PR diff reflects the fixes.

## Output
A table:
```
| # | Source (Sweeper/teknium/CI) | Concern | Gate | Status | Action |
```
Plus a one-line verdict: READY (all addressed) / NEEDS WORK (N items open).

## Common Pitfalls
1. **Sweeper labels are heuristics** — don't rewrite working code just to satisfy a tag.
2. **Maintainer inline comments beat bot labels** — prioritize those.
3. **Force-push without consent** — ask before `--force-with-lease`.
4. **Stale diff** — re-validate against the REAL current diff
   (`git diff UPSTREAM/main...HEAD`), not memory.
5. **`gh` not on MSYS PATH (Windows)** — use the full path.

## Verification Checklist
- [ ] All review items pulled and classified (Sweeper / maintainer / CI).
- [ ] Each item has a gate-checked verdict.
- [ ] Approved fixes applied and re-validated via `hermes-assurance-gate`.
- [ ] Branch pushed fast-forward (or force only after consent).
- [ ] Review-summary comment posted; PR diff reflects fixes.
