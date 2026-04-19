# extended-rca

> **Stop writing shallow postmortems.** A Claude Code plugin that runs a thorough, methodology-aware root cause analysis on resolved software incidents — and produces a structured engineering-grade postmortem that resists the usual RCA failure modes.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-8A2BE2)](https://docs.claude.com/en/docs/claude-code)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](./CONTRIBUTING.md)

---

## The problem

Most RCAs stop at the first plausible cause. The team finds a deploy that introduced a bug, writes a one-ticket postmortem, and moves on. Six weeks later, the same class of incident recurs — because "the deploy" was a trigger, not a root cause, and nobody looked at the contributing factors or the systemic conditions that made the trigger possible.

This plugin resists that pull. It gives Claude Code a disciplined workflow — 5-whys that don't stop at "human error," fishbone-style hypothesis generation across multiple categories, explicit layer separation (trigger / proximate / contributing / systemic), corrective actions tagged by type, and a self-check pass that catches hedged findings before handing the artifact over. The output is a postmortem your team can actually learn from.

## Quick start

```bash
# 1. Clone into your Claude Code plugins directory
git clone https://github.com/sebdenes/ExtendedRCA ~/.claude/plugins/extended-rca

# 2. Restart Claude Code. Then in any project:
/rca we had a 45-minute checkout outage starting 14:02 UTC, 17% of requests failed...
```

That's it. Claude reads the skill, picks the right mode for your inputs, and produces a structured postmortem Markdown file.

## What the skill actually does

Five modes, picked automatically or explicitly:

| Mode | When to use | What you get |
|---|---|---|
| `/rca <summary>` | Default. You describe the incident or paste a timeline. | Full extended RCA with fishbone hypothesis generation, a 5-whys chain done properly, layered findings, and tagged actions. |
| `/rca deepen <draft>` | You already have a postmortem that stops too shallow. | Your draft + added alternative hypotheses, contributing factors, systemic causes, and tagged corrective actions. |
| `/rca review <postmortem>` | You just wrote one and want it graded, not rewritten. | A findings list scoring 10 axes out of 30, with specific issues cited and a recommended next step (share / deepen / rewrite). |
| `/rca hypotheses <summary>` | You want help thinking broadly before committing to a story. | At least three candidate hypotheses, fishbone-categorized, with confirm/disconfirm evidence notes. |
| `/rca trends <folder>` | You have N postmortems and want recurring systemic themes identified. | A synthesis document with 3–5 recurring themes, evidence counts, and recommended org-level interventions. |

## Why this resists real failure modes

| Common RCA failure | How extended-rca resists it |
|---|---|
| Single-cause narrative ("the deploy broke it") | Forces multi-hypothesis generation across 6 fishbone categories (People / Process / Code / Infrastructure / Data / External) |
| Why-chain ends at "human error" | The skill and `references/five-whys.md` explicitly reject human error as a terminal cause |
| Layer confusion (trigger/proximate/systemic smooshed into "root cause") | The artifact template has separate sections for each layer |
| Action list is all "Prevent" | Actions are tagged Prevent / Detect / Mitigate / Respond; a healthy set spans ≥3 buckets |
| Hedged systemic findings ("communication could be improved") | Phase 6 self-check catches hedged phrasing and forces specific, non-blameful rewrites |
| Blame-ful phrasing toward a person or team | Self-check catches it before handover; the cross-team example shows the blameless-but-specific pattern |
| No record of rejected hypotheses | A dedicated "Hypotheses considered and ruled out" section |
| Postmortem reads as purely damning | A "What went well" section to preserve the patterns that worked |
| Single-incident tunnel vision | `/rca trends` surfaces recurring systemic themes across a folder of postmortems |
| Silent regression of skill quality | `evals/` contains a structured suite of cases and a rubric for catching regressions |

## What you get in the output

A Markdown postmortem with deliberately-structured sections (see [`skills/extended-rca/references/artifact-template.md`](./skills/extended-rca/references/artifact-template.md)):

- **Summary** — one paragraph the executive team can read
- **Impact** — measurable, with blast radius
- **Timeline** — tight, timestamped
- **Trigger → Proximate → Contributing → Systemic** — the four causal layers, kept separate
- **Hypotheses considered and ruled out** — with the disconfirming evidence
- **What went well** — preserve what worked
- **Corrective actions** — tagged Prevent / Detect / Mitigate / Respond, with owners
- **Open questions** — honest about what the evidence didn't support

## Worked examples

Five shallow/deepened pairs under [`examples/`](./examples), each exercising a different fishbone category and failure mode:

| Example | Class | Key teaching point |
|---|---|---|
| [`login-outage`](./examples/login-outage) | Code / Process | The canonical shallow draft; cross-service review gap |
| [`capacity-surge`](./examples/capacity-surge) | Infrastructure / Process | Capacity failures aren't knob problems — they're planning problems |
| [`silent-data-corruption`](./examples/silent-data-corruption) | Data / People | Ownership ambiguity between data and ML teams |
| [`third-party-timeout`](./examples/third-party-timeout) | External / Code | External trigger ≠ exempt from analysis |
| [`cross-team-breaking-change`](./examples/cross-team-breaking-change) | Process / People | Blameless-but-specific reframing of a blame-ful draft |

Try one:

```bash
# In Claude Code, from this repo's root:
/rca deepen examples/capacity-surge/shallow-draft.md
```

Compare Claude's output against `deepened.md`. The interesting differences are in the contributing-factors and systemic-causes layers — the ones shallow drafts almost always miss.

## Evals

The skill ships with a small eval suite in [`evals/`](./evals). Six cases covering the review, deepen, hypotheses, and full-RCA modes, graded against a 15-point rubric for postmortems, a 5-point rubric for reviews, and a 4-point rubric for hypotheses output. Run before merging any change to `SKILL.md`, `commands/rca.md`, or the references — it catches the prompt regressions that silently reintroduce shallow output. See [`evals/README.md`](./evals/README.md) for the protocol.

## Layout

```
extended-rca/
├── .claude-plugin/plugin.json
├── commands/
│   └── rca.md                           # /rca slash command
├── skills/extended-rca/
│   ├── SKILL.md                         # philosophy, phases, anti-patterns
│   └── references/
│       ├── five-whys.md                 # 5-whys without the usual traps
│       ├── fishbone.md                  # 6 software-specific categories
│       ├── artifact-template.md         # the postmortem template
│       ├── self-check.md                # Phase 6 verification checklist
│       ├── review-rubric.md             # /rca review grading rubric
│       └── trends-synthesis.md          # /rca trends guide
├── examples/
│   ├── login-outage/                    # Code / Process
│   ├── capacity-surge/                  # Infrastructure / Process
│   ├── silent-data-corruption/          # Data / People
│   ├── third-party-timeout/             # External / Code
│   └── cross-team-breaking-change/      # Process / People
├── evals/
│   ├── README.md                        # why and when to run
│   ├── rubric.md                        # the graded checks
│   ├── cases.md                         # the case list
│   └── run.md                           # execution protocol
└── .github/                             # issue / PR templates
```

## Installation

```bash
git clone https://github.com/sebdenes/ExtendedRCA ~/.claude/plugins/extended-rca
# Restart Claude Code. The /rca command is now available.
```

## Contributing

PRs welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md). Good first issues: additional (anonymized) incident case studies, new eval cases, refinements to the 5-whys and fishbone references, improvements to the artifact template or review rubric.

## License

MIT — see [LICENSE](./LICENSE).

## If this saved you from shipping a shallow postmortem

Star the repo. That's the whole ask.
