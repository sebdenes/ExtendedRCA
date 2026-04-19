# Eval cases

Each case specifies an input file, an invocation, and the expected structural properties. Cases are sourced from the `examples/` folder to keep the suite grounded in real example data.

Score each case pass / partial-pass / fail using the rubric in `rubric.md`.

## Case 1 — Review a shallow draft (login outage)

- **Input**: `examples/login-outage/shallow-draft.md`
- **Invocation**: `/rca review examples/login-outage/shallow-draft.md`
- **Expected**:
  - R1–R5 all pass (rubric.md, Review-output checks).
  - Grade ≤ 12/30. The draft is deliberately shallow; a grade above 12 indicates the review is grading on style rather than rigor.
  - Must flag: missing hypotheses section (axis 2), no layer separation (axis 4), single-bucket action list (axis 5), vague language (axis 7).
  - Recommended next step: "Run `/rca deepen`."
- **Regression indicator**: grade > 14, or any of the four must-flag issues is missed.

## Case 2 — Review a deepened postmortem (login outage)

- **Input**: `examples/login-outage/deepened.md`
- **Invocation**: `/rca review examples/login-outage/deepened.md`
- **Expected**:
  - R1–R5 all pass.
  - Grade ≥ 25/30.
  - "Specific issues" section is short or empty — the deepened postmortem is near-exemplar.
  - Recommended next step: "This postmortem is ready to share."
- **Regression indicator**: grade < 22, or the review invents issues that aren't there.

## Case 3 — Deepen a shallow draft (capacity surge)

- **Input**: `examples/capacity-surge/shallow-draft.md`
- **Invocation**: `/rca deepen examples/capacity-surge/shallow-draft.md`
- **Expected**:
  - The output is a new postmortem (not a rewrite of the shallow draft in place — make the diff visible).
  - P1–P15 all pass (rubric.md, Postmortem-output checks), scoring at least 12/15.
  - P6: at least 2 contributing factors.
  - P7: at least 1 systemic cause that names a structural condition (capacity-planning ownership, organizational gap — not just "we should test more").
  - P10: actions span at least 3 of the 4 taxonomy buckets.
  - P12: why-chain does not terminate at "the pool was too small."
- **Regression indicator**: scoring below 10/15, or the output is a minor edit of the shallow draft rather than a substantive deepening.

## Case 4 — Deepen a cross-team incident (breaking change)

- **Input**: `examples/cross-team-breaking-change/shallow-draft.md`
- **Invocation**: `/rca deepen examples/cross-team-breaking-change/shallow-draft.md`
- **Expected**:
  - P13 (no blame-ful phrasing) passes — the shallow draft is itself blame-ful ("auth team shouldn't have..."), and the deepened output must reframe to system/process level.
  - P7: systemic cause identifies the absence of a deprecation policy or similar structural gap, not "teams should communicate more."
  - P10: actions span at least 3 buckets, including a Respond-tagged action.
- **Regression indicator**: blame-ful phrasing preserved or amplified, or systemic cause reads as "people should try harder."

## Case 5 — Hypotheses only (third-party timeout)

- **Input**: The summary from `examples/third-party-timeout/shallow-draft.md` plus the timeline.
- **Invocation**: `/rca hypotheses <summary pasted inline>`
- **Expected**:
  - H1–H4 all pass (rubric.md, Hypotheses-output checks).
  - At least 3 hypotheses, categorized across fishbone buckets.
  - Does not jump to "Stripe was slow" as the conclusion; treats that as one hypothesis among several.
- **Regression indicator**: fewer than 3 hypotheses, or a single-hypothesis conclusion despite the invocation being `hypotheses` mode.

## Case 6 — Full RCA from a fresh narrative (silent data corruption)

- **Input**: the summary and impact paragraphs from `examples/silent-data-corruption/shallow-draft.md` — treat as a *fresh* incident, not a draft to deepen.
- **Invocation**: `/rca <summary pasted inline>`
- **Expected**:
  - P1–P15 with scoring at least 12/15.
  - P8: at least 3 hypotheses considered (this tests default-mode phase 2).
  - P12: why-chain reaches a systemic cause (data-quality ownership) rather than terminating at "the migration had a bug."
  - Output is saved to a named file with a clear path.
- **Regression indicator**: the output collapses to a one-page "the migration was buggy" summary, or stops at 1–2 hypotheses.

## Running the suite

See `run.md` for the execution protocol. Document results in a results file (e.g. `results-2026-04-19.md`) with grades and any regression notes.
