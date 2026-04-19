# Eval rubric

A set of structural checks to apply to any postmortem or review output produced by the skill. Use these checks to grade eval-case outputs and to catch regressions.

## Postmortem-output checks

Applied to any artifact produced by `/rca`, `/rca deepen`, or `/rca hypotheses` in "full" mode.

| # | Check | Pass criterion |
|---|---|---|
| P1 | Facts present | Summary, Impact, Timeline sections exist and are non-empty |
| P2 | Impact quantified | At least one measurable number (duration, percentage, count, revenue) in Impact |
| P3 | Timeline is timestamped | At least 3 events with explicit times |
| P4 | Trigger is a single concrete event | Trigger section names one event, not a list |
| P5 | Proximate cause is specific | Names services, endpoints, configs, or code paths; not "a bug" |
| P6 | Contributing factors ≥ 2 | At least two distinct contributing factors listed |
| P7 | Systemic cause ≥ 1 | At least one systemic / latent cause that names a structural, non-individual condition |
| P8 | Hypotheses ≥ 3 | At least three hypotheses considered (in full-mode output; exempt in `deepen` mode if input had 0 and deepen adds 2) |
| P9 | Ruled-out hypotheses have disconfirming evidence | Each ruled-out hypothesis has a one-line reason |
| P10 | Actions span ≥ 3 buckets | At least 3 of Prevent / Detect / Mitigate / Respond have content |
| P11 | Every action has an owner | No action tagged "Owner: ???" or missing entirely |
| P12 | No terminal human-error cause | Why-chain does not end at "person made a mistake" or a single commit |
| P13 | No blame-ful phrasing | No phrases like "X should have," "the team was careless," "[person] failed to" |
| P14 | No hedged systemic findings | No phrases like "communication could be improved," "we should perhaps consider" |
| P15 | "What went well" is specific | Names concrete patterns, not generic praise |

Score: pass = 1, partial = 0.5, fail = 0. Out of 15. Acceptable floor: 12/15. Strong baseline: 14/15.

## Review-output checks

Applied to artifacts produced by `/rca review`.

| # | Check | Pass criterion |
|---|---|---|
| R1 | Grade is numeric and in range | X/30 where X is 0–30 |
| R2 | All 10 axes scored | Table has 10 rows with a 0–3 score each |
| R3 | Specific issues cited | Each issue quotes or paraphrases a specific passage |
| R4 | Recommended next step is one of the three | Ready / Deepen / Rewrite |
| R5 | No rewriting of the source | Review does not produce a fixed postmortem |

Score: pass = 1, fail = 0. Out of 5. All checks must pass.

## Trends-output checks

Applied to artifacts produced by `/rca trends`.

| # | Check | Pass criterion |
|---|---|---|
| T1 | Source count reported | N postmortems consumed, explicit |
| T2 | At least one theme with ≥ 2 evidence | Top theme cites at least two source incidents |
| T3 | Themes grounded in specific evidence | Each theme names systems, teams, or processes, not generic categories |
| T4 | Recommendations are unhedged | No "perhaps consider" in recommendation lines |
| T5 | Meta-finding if present is noted | If the corpus reveals a pattern about the postmortem process itself, it's called out |

Score: pass = 1, fail = 0. Out of 5. Pass floor: 4/5.

## Hypotheses-output checks

Applied to artifacts produced by `/rca hypotheses`.

| # | Check | Pass criterion |
|---|---|---|
| H1 | ≥ 3 hypotheses | Output has at least three distinct hypotheses |
| H2 | Fishbone-categorized | Each hypothesis is tagged with one of the 6 fishbone categories |
| H3 | Confirm/disconfirm notes | Each hypothesis has at least one note on confirming or disconfirming evidence |
| H4 | No premature convergence | Output does not declare a single winner |

Score: pass = 1, fail = 0. Out of 4. All must pass.
