---
name: hermes-bug-report
description: Write a Hermes Agent bug report with a premise check.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes-agent, bug-report, contribution, triage]
    related_skills: [hermes-feature-request, hermes-bug-feature-reporting, hermes-contributing-check]
---

# Hermes Bug Report

Produce a consistent, repo-compliant bug report for Hermes Agent. The report is
DATA until the user says otherwise — do not commit or open an issue unless asked.
Separate from `hermes-feature-request` (which proposes new capability) and from
`hermes-bug-feature-reporting` (which adds Discord triage + a hard verification
gate). Use this skill for a clean, template-driven bug write-up.

## Overview

This skill focuses on the *write-up*: a structured bug report plus a premise
check that stops you from filing non-bugs (documented behavior, already-fixed
issues, stale installs). It does not enforce Discord-first triage — if you want
that process, use `hermes-bug-feature-reporting`.

## When to Use
- User wants to report a Hermes Agent bug and needs a consistent structure.
- After confirming a real defect and wanting a clean GitHub-ready write-up.

Don't use for:
- Feature requests (use `hermes-feature-request`).
- Full triage with Discord maturation (use `hermes-bug-feature-reporting`).

## Sources of truth
- `CONTRIBUTING.md` (repo root) — process, scope, style.
- `AGENTS.md` (repo root) — design intent, "Before you call it a bug".
- Repo: `NousResearch/hermes-agent` (upstream); local checkout at the user's path.

## Step 1 — Premise check (AGENTS.md "Before you call it a bug")
Verify the premise BEFORE writing the report. If any fails, say so and STOP:
- Documented/intended behavior? Grep `AGENTS.md` + `CONTRIBUTING.md`.
- Already fixed on `upstream/main`? `git fetch upstream && git log --oneline upstream/main -S "<symbol>"`.
- Stale venv / partial install? Re-run the installer; check the venv.

Completion: all three checks resolved as "not a non-bug", or the user explicitly
overrides with evidence.

## Step 2 — Fill the template
```
# Bug: <one-line summary>

## Environment
- Hermes version / commit: <hermes --version or git rev-parse HEAD>
- Platform: Windows / macOS / Linux (distro + version)
- Install type: installer / venv / source
- Provider/model in use:

## Symptom
Verbatim error / traceback (paste the real output, do not paraphrase).

## Expected
What should have happened.

## Reproduction
1. <exact steps>   (minimal repro preferred; note config.yaml / .env / plugins)

## Premise check
- Documented? <yes/no + where>
- Fixed on upstream/main? <yes/no + commit>
- Stale install? <yes/no>

## Impact / Blast radius
Who affected, severity, workaround.

## Suggested fix (optional)
Likely file/function.
```

Completion: every section filled with real data; Symptom is a verbatim traceback.

## Deliverable
The filled report text. Offer to save it to a file or open a GitHub issue only if asked.

## Common Pitfalls
1. **Filing a non-bug** — the premise check exists to stop this; honor it.
2. **Invented tracebacks** — paste real output; fabricated error text wastes maintainer time.
3. **Backticks in `git commit -m`** — command substitution on bash; use straight quotes.
4. **Hard-coded local paths** — keep the template portable; user supplies their own paths.

## Verification Checklist
- [ ] Premise check passed (documented? / fixed? / stale install?).
- [ ] Environment section has Hermes version + platform + install type.
- [ ] Symptom is a verbatim traceback, not paraphrased.
- [ ] Reproduction is step-by-step and minimal.
- [ ] Report is offered (not auto-committed) unless the user asked to file it.
