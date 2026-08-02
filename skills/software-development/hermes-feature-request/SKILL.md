---
name: hermes-feature-request
description: Write a Hermes Agent feature request with scope decision.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes-agent, feature-request, contribution, design]
    related_skills: [hermes-bug-report, hermes-contributing-check, hermes-bug-feature-reporting]
---

# Hermes Feature Request

Produce a consistent, repo-compliant feature request for Hermes Agent. DATA
until the user says otherwise — do not implement or open an issue unless asked.
Separate from `hermes-bug-report` (defect write-up) and from
`hermes-bug-feature-reporting` (which adds Discord triage). This skill focuses on
the structured proposal: footprint ladder, scope decision, design constraints.

## Overview

This skill is the *write-up* for a new capability: it forces an explicit scope
decision (Skill / Tool / Plugin) and cites the Hermes design rules (Footprint
Ladder, prompt-cache sacredness, narrow waist). It does not implement anything.

## When to Use
- User wants to propose a Hermes Agent feature and needs a consistent structure.
- Before opening a feature issue or a PR, to pre-align with maintainer expectations.

Don't use for:
- Bug reports (use `hermes-bug-report`).
- Full Discord-triage process (use `hermes-bug-feature-reporting`).

## Sources of truth
- `CONTRIBUTING.md` — "Should it be a Skill or a Tool?", "What we don't want".
- `AGENTS.md` — "The Footprint Ladder", "What we want / don't want".
- Developer Guide: https://hermes-agent.nousresearch.com/docs/developer-guide
  - adding-tools (tools vs skills; handler returns JSON; errors as `{"error": ...}`; check_fn; plugin preferred)
  - creating-skills (SKILL.md format)
  - context-compression-and-caching (prompt-cache sacredness)

## Template
```
# Feature: <one-line summary>

## Problem
User pain; link related bug/issue.

## Proposal
Concrete behavior; CLI flag / config key / skill / tool? Show invocation + example output.

## Footprint Ladder (AGENTS.md)
Rung chosen + why a core tool is/isn't justified:
1 extend existing -> 2 CLI cmd + skill -> 3 service-gated tool (check_fn)
-> 4 plugin -> 5 MCP server -> 6 new core tool (last resort).

## Scope decision (CONTRIBUTING.md / dev-guide/adding-tools)
- Skill: instructions + shell + existing tools, no new model-tool surface.
- Tool: needs API keys, custom processing, binary/streaming handling.
- Plugin: third-party product / standalone integration.
Pick one, cite the rule.

## Design constraints (AGENTS.md + dev-guide/context-compression-and-caching)
- Per-conversation prompt caching is SACRED — no mid-conversation system-prompt
  rebuild, no toolset swap, no past-context mutation, no injected synthetic user msg.
- Core is a narrow waist; capability at the edges.
- Behavior contracts over snapshots (tests assert invariants, not frozen values).

## Alternatives considered
What was rejected + why.

## Test plan
Unit + GitHub Actions parity command.
```

Completion: every section filled; scope decision cites CONTRIBUTING.md or the
dev-guide; design constraints reference the prompt-cache rule.

## Deliverable
Filled feature text. Offer to save or open an issue only if asked.

## Common Pitfalls
1. **Core tool without justification** — justify why terminal+file / a skill won't do.
2. **Config in .env** — secrets only; everything else goes in config.yaml.
3. **Ignoring the prompt-cache rule** — any design that rebuilds the system prompt
   mid-conversation will be rejected.
4. **Vague proposal** — show the concrete invocation + example output.

## Verification Checklist
- [ ] Problem stated with linked pain / related issue.
- [ ] Footprint Ladder rung chosen + justified.
- [ ] Scope decision (Skill/Tool/Plugin) made with a cited rule.
- [ ] Design constraints reference prompt-cache sacredness + narrow waist.
- [ ] Alternatives considered + Test plan present.
- [ ] Report offered (not auto-filed) unless the user asked.
