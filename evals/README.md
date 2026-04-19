# Evals — extended-rca

A small evaluation harness for the `extended-rca` skill. The goal is to catch regressions when the skill's prompts, references, or workflow change. If a PR modifies `SKILL.md`, `commands/rca.md`, or any file in `references/`, it should be checked against this suite before merging.

The harness is deliberately markdown-only — no Python, no test runner. Each case is an input document, an invocation, and a set of expected structural properties. Claude Code runs the invocation and grades itself against the expectations. This is light enough to run by hand and structured enough to catch the regressions that actually matter.

## Why this exists

The failure modes this skill is designed to resist — single-cause narratives, terminal human-error causes, hedged systemic findings, all-Prevent action lists — are the same failure modes that small prompt changes can silently reintroduce. Without evals, a "clarity improvement" to `SKILL.md` can quietly degrade output quality and no one notices until an actual postmortem comes out shallow.

Evals give you a floor. They don't prove the skill is good; they prove it hasn't regressed against a known-good baseline.

## How to use

See `run.md` for the step-by-step protocol. Summary:

1. For each case listed in `cases.md`, invoke the specified command against the input.
2. Compare the output against the expected properties.
3. Score each case pass / partial-pass / fail.
4. If any case regresses from the baseline, the change requires explicit justification.

## Case coverage

The current case set exercises:
- `/rca review` on shallow drafts (should grade low, flag specific issues)
- `/rca review` on deepened postmortems (should grade high)
- `/rca deepen` on shallow drafts (output should score as acceptable or strong via review)
- `/rca hypotheses` on a fresh incident (should produce at least 3 fishbone-categorized hypotheses)

Cases are sourced from the `examples/` folder. Adding a case means adding an example, so the incident diversity of the suite grows with the incident diversity of the examples.

## When evals should fail

Evals should fail if any of the following regresses:
- `/rca review` on a known shallow draft grades it above 14/30
- `/rca review` on a known deepened postmortem grades it below 22/30
- `/rca deepen` output is missing any of the four causal layers
- `/rca deepen` output has an action list tagged exclusively Prevent
- Any workflow produces a postmortem with a terminal "human error" cause in the why-chain
- Any workflow produces a postmortem with blame-ful phrasing toward a named person or team

A regression on any of these is a signal that the skill has drifted away from its design goals, regardless of whether the change looked like a clarity improvement.

## Scoring variance

LLM outputs have variance. A single eval run is noisy — a case might fail once and pass on re-run. For baseline certification of a change, run each case 3 times; require 2 of 3 passes. Document the variance in the PR.

## Adding a case

A new case needs:
- A source input (ideally an existing file in `examples/`).
- An invocation (`/rca <mode> <path>`).
- Expected properties (structural, not textual — the skill should produce output with *properties*, not with *exact phrasing*).
- An entry in `cases.md`.

Don't over-constrain. Expected properties should test what the skill must do, not what it might incidentally do.
