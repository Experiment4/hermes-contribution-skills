---
name: hermes-contributing-check
description: Check Hermes Agent diffs for contributing compliance.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes-agent, contribution, code-review, architecture]
    related_skills: [hermes-assurance-gate, hermes-pr-feedback-loop, contributing-review, requesting-code-review]
---

# Hermes Contributing Check

Run the full static-review gauntlet on a Hermes Agent diff/PR: contributing
guidelines, architecture/design patterns, and code best-practices. Produces a
list of merge-blockers and design-blockers with the exact policy cited.

## Overview

This skill wraps `contributing-review` and `requesting-code-review` with
Hermes-specific policy grounded in `CONTRIBUTING.md`, `AGENTS.md`, and the
Developer Guide. It is the analysis half of the gate; the enforcement half
(compile/test/push) lives in `hermes-assurance-gate`. Every finding cites a real
source — no invented policy.

## When to Use
- After implementing a Hermes Agent change, before opening or updating a PR.
- User says "check the diff", "review against contributing", "architecture check".

Don't use for:
- Non-Hermes repos (use the generic `contributing-review` / `requesting-code-review`).
- Doc-only changes where architecture/test checks are irrelevant (skip Step 3).

## References
- `references/dev-guide-rules.md` — how to navigate the dev-guide (llms.txt index;
  bare /docs/developer-guide 404s) + the blocking Hermes rules (prompt-cache sacred,
  tool handler contract, footprint ladder) condensed from AGENTS.md / dev-guide.

## Sources of truth
- `CONTRIBUTING.md` (repo root, ~1009 lines) — process, scope, style, tool/skill rules.
- `AGENTS.md` (repo root, ~1435 lines) — "What we want / don't want", "The Footprint Ladder", prompt-cache rule.
- Developer Guide: https://hermes-agent.nousresearch.com/docs/developer-guide
  - context-compression-and-caching (prompt caching sacred; cache-aware patterns)
  - adding-tools (tools vs skills; handler returns JSON; errors as {"error": ...}; check_fn; plugin preferred)
  - creating-skills, agent-loop, prompt-assembly, gateway-internals

## Step 1 — Contributing-guidelines check (merge-blockers)
Apply the 8 steps of `skill_view(name="contributing-review")` to the diff. Enforce:
- Conventional Commits `<type>(<scope>): <desc>` with real scope (cli, gateway, logging, desktop, tui, ...).
- PR description: what/why, how to test, platforms tested, issue refs.
- Changes limited to PR scope (no drive-by refactors).
- No hardcoded paths — use `get_hermes_home()` / `HERMES_HOME`.
- Cross-platform: no Windows-only bashisms, no unguarded `os.system`.
- Docs updated for user-facing changes (README / docs/user-guide / cli-config examples).
- Logging format `YYYY-MM-DDTHH:MM:SS.sssZ|LEVEL|LOGGER|PID|TID|MESSAGE`.
- No secrets in diff. One logical change per PR.

## Step 2 — Architecture / design-pattern check (design-blockers)
Read AGENTS.md § "What we want / What we don't want" + § "The Footprint Ladder", plus dev-guide pages. Verify:
- Extends edges, not the core waist, unless justified.
- Does NOT break prompt-cache stability (no mid-convo system-prompt change; no
  alternation break — never two same-role messages in a row; no synthetic user
  message injected mid-loop). Cite dev-guide/context-compression-and-caching.
- Reuses existing infra instead of duplicating ("Extend, don't duplicate").
- Cache-/invariant-safe tests (behavior contracts, not change-detectors).
- Tools: handler returns JSON string; errors as `{"error": ...}`; `check_fn` gates
  availability; plugin route preferred over core tool (dev-guide/adding-tools).
- Plugins: own dir, use ABCs/hooks, no core-file edits.
- New config in config.yaml, NOT .env (secrets only).
Report each violation with the §/page cited.

## Step 3 — Best-practice / code-quality check
Run `skill_view(name="requesting-code-review")` (Steps 1-8): diff capture, static
security scan (secrets, shell injection, eval/exec, pickle, SQLi), baseline-aware
test+lint (pytest; ruff/mypy if present), self-review checklist, independent
reviewer subagent via `delegate_task`. Apply `hermes-agent-contrib` hard rules:
- No import-time side effects in `hermes_cli/*` (logic inside the called function).
- Reuse `_m()._detect_concurrent_hermes_instances(scripts_dir)`; never raw `tasklist`
  by image name (wrong-install / PID-1 risk).
- Read parsed `args`, not `sys.argv`.
- Y/n prompt: `n` -> clean `sys.exit(0)`; `y` -> `taskkill /PID <pid> /F`.

## Output
A report with three sections: MERGE-BLOCKERS, DESIGN-BLOCKERS, QUALITY-FINDINGS
(non-blocking). Each item cites the source (CONTRIBUTING.md §, AGENTS.md §, or
dev-guide page). Do not auto-fix; report and let the user decide.

## Common Pitfalls
- Don't invent policy — every finding cites a real source.
- For doc-only / pure-config changes, skip Step 3.

## Verification Checklist
- [ ] MERGE-BLOCKERS listed (Conventional Commits scope, PR desc, no hardcoded paths, cross-platform, docs, no secrets).
- [ ] DESIGN-BLOCKERS listed (edges not core waist, prompt-cache stable, no duplication, config.yaml not .env).
- [ ] QUALITY-FINDINGS listed (static scan + reviewer subagent; hermes-agent-contrib hard rules applied).
- [ ] Every finding cites CONTRIBUTING.md § / AGENTS.md § / dev-guide page.
- [ ] No auto-fix applied; user decides next step.
