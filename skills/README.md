# Skills

Skills extracted from the **Tech Writer's Tribe** lessons (Modules 1–4). Each file is a reusable prompt — copy it into `.claude/commands/` to run it as `/<name>`, or run it directly from here.

`$ARGUMENTS` is the file path you pass when you invoke the skill, e.g. `/m3-check-api-doc sample-docs/orders-api.md`.

| Skill | Module | Pattern | Purpose |
|-------|--------|---------|---------|
| `m1-completeness-check` | 1 | Reviewer | Mark each required API section PRESENT / INCOMPLETE / MISSING |
| `m1-doc-feedback` | 1 | Reviewer | Colleague-style feedback: works / unclear / 3 improvements |
| `m2-skill-or-tool` | 2 | Analyzer | Classify tasks as Skill, Tool, Both, or Neither |
| `m3-check-api-doc` | 3 | Reviewer | Full RTCCO completeness review of an API doc |
| `m4-check-style` | 4 | Reviewer | Check a doc against the team style guide |
| `m4-release-notes` | 4 | Generator | Generate release notes from a changelog or git log |
| `m4-simplify-prose` | 4 | Transformer | Rewrite expert prose for a beginner audience |
| `m4-doc-coverage` | 4 | Analyzer | Surface documentation gaps and undocumented parameters |
| `watch-twtai` | — | Utility | Sync new/updated skills from the `AmanProjects/TWTAI` repo into `.claude/commands/` |

## Try them on the sample doc

```
/m3-check-api-doc sample-docs/orders-api.md
/m4-check-style    sample-docs/orders-api.md
/m4-doc-coverage   sample-docs/orders-api.md
/m4-simplify-prose sample-docs/orders-api.md
```

## Staying in sync

New skills published to the `skills/` folder of the [`AmanProjects/TWTAI`](https://github.com/AmanProjects/TWTAI) repo are pulled into `.claude/commands/` by the `/watch-twtai` skill.
