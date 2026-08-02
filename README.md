# Hermes Contribution Skills

A skill set that runs the full Hermes Agent contribution lifecycle with
consistent, repo-compliant output:

| Skill | Purpose |
|-------|---------|
| `hermes-bug-report` | Consistent bug report + premise check (documented? / fixed? / stale install?). |
| `hermes-feature-request` | Structured feature proposal: footprint ladder + scope decision (Skill/Tool/Plugin). |
| `hermes-contributing-check` | Static gauntlet: contributing + architecture + best-practice, cites CONTRIBUTING.md / AGENTS.md / dev-guide. |
| `hermes-assurance-gate` | Pre-PR local quality gate: compile, import side-effect, tests, clean tree, push. |
| `hermes-open-pr` | Open the PR via your fork after the gate passes. |
| `hermes-pr-feedback-loop` | Process Sweeper/maintainer review feedback and re-validate the gates. |

## Lifecycle

```
hermes-feature-request (or hermes-bug-report)
  -> implement
  -> hermes-contributing-check
  -> hermes-assurance-gate
  -> hermes-open-pr
  -> hermes-pr-feedback-loop   (on review feedback)
  -> back to contributing-check / gate
```

## Install

```
hermes skills install hermes-contribution-skills
```

Or copy `skills/software-development/<name>/` into your `~/.hermes/skills/`.

## License

MIT
