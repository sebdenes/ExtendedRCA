# Review rubric — `/rca review <path>`

In review mode, you grade an existing postmortem against the quality rubric **without rewriting it**. The output is a findings document. The user decides what to do with the findings — in particular, whether to invoke `/rca deepen` to fix what's broken.

This mode exists because the most common real-world scenario isn't "generate from scratch" or "silently fix" — it's "I just wrote this, is it any good?" A review gives structured feedback that lets the author keep ownership.

## Scoring axes

Score each axis on a 0–3 scale:

- **0 — Missing.** The section is absent or empty.
- **1 — Present but shallow.** Content exists but misses the bar for the axis (e.g. single-cause narrative, all-Prevent action list, hedged systemic findings).
- **2 — Acceptable.** Content meets the bar but has room to go deeper.
- **3 — Strong.** Content is specific, rigorous, and useful as-is.

### Axes

1. **Facts established** — impact, timeline, blast radius. Measurable. Specific.
2. **Hypothesis breadth** — at least three considered, with confirming/disconfirming evidence noted. Alternatives explicitly ruled out.
3. **Why-chain depth** — chains terminate at a structural cause, not at "human error" or a single commit.
4. **Layer separation** — trigger / proximate / contributing / systemic are distinct sections with real content in each.
5. **Action taxonomy coverage** — actions tagged Prevent / Detect / Mitigate / Respond, spanning at least three buckets.
6. **Action-to-finding traceability** — every action traces to a specific contributing or systemic cause.
7. **Language specificity** — services, endpoints, configs, flags named; no "the system broke" filler.
8. **Blamelessness** — no person-blaming or team-blaming phrasing; findings point at systems.
9. **"What went well" is real** — specific patterns named, not generic praise.
10. **Honest about uncertainty** — weak findings live in "open questions," not stated as fact.

Sum the axis scores for a /30 grade. Anything under 18 is shallow; 18–24 is acceptable; 25+ is strong.

## Review output format

Save the findings to `./review-<postmortem-filename>.md`. Use this structure:

```markdown
# Review of <postmortem name>

**Grade: <N>/30** — <shallow / acceptable / strong>

## Axis scores

| Axis | Score | Note |
|---|---|---|
| Facts established | N | <one-line assessment> |
| Hypothesis breadth | N | <one-line assessment> |
| Why-chain depth | N | <one-line assessment> |
| Layer separation | N | <one-line assessment> |
| Action taxonomy coverage | N | <one-line assessment> |
| Action-to-finding traceability | N | <one-line assessment> |
| Language specificity | N | <one-line assessment> |
| Blamelessness | N | <one-line assessment> |
| "What went well" is real | N | <one-line assessment> |
| Honest about uncertainty | N | <one-line assessment> |

## Specific issues

<Use this section to cite concrete problems by quoting or paraphrasing passages from the postmortem. Each issue should be:
- Locatable (quote the offending passage or name the section)
- Specific (name the axis and the failure pattern)
- Actionable (suggest the direction of a fix — not the fix itself)

Example: 
- **Hedged systemic finding (Axis 3, 10).** The "Root causes" section says "communication between teams could be improved." This is a hedged finding — it names no teams, no specific gap, no evidence, and is not actionable. A review-level fix would replace it with a specific claim: which teams, what channel doesn't exist, what evidence supports the claim that it's systemic. Currently this reads as filler.>

## What this postmortem does well

<Don't make this section up to be nice. Only cite axes that genuinely scored 2 or 3. If the postmortem is shallow across the board, write "No axes scored above shallow; recommend a substantial rewrite."

Example:
- The timeline is tight and well-timestamped. Detection, escalation, mitigation, and resolution events are all present with minute-level precision.
- Impact is quantified (17% of requests failed, 45-minute duration, X dropped orders).>

## Recommended next step

<Pick one:
- **This postmortem is ready to share.** No substantive changes needed. (Grade 25+)
- **Run `/rca deepen` on this file.** The missing content is in the contributing-factors and systemic-causes layers. (Typical grade 14-22)
- **Rewrite from scratch.** The factual base is too weak to deepen; start over with `/rca` using the original incident timeline. (Grade under 14 or structurally confused)

Give the reason for your choice in one sentence.>
```

## Things to avoid in review mode

- **Don't rewrite the postmortem.** If you find yourself producing a fixed version, you're in `deepen` mode. Review returns *findings*, not a rewrite.
- **Don't grade on style alone.** A well-written shallow postmortem and a clunky rigorous one should score very differently. The rubric prioritizes rigor over polish.
- **Don't inflate scores to be kind.** A postmortem that scores 12/30 should be told it scored 12/30. Softening the grade defeats the point of reviewing.
- **Don't score axes the postmortem didn't try to cover.** If there's no "what went well" section, that's a 0 on Axis 9, not an exemption.

## When to suggest `/rca deepen` vs. rewriting

Deepen works well when the factual base is sound but the analysis is shallow — impact, timeline, and trigger are right, but contributing factors and systemic causes are thin or missing. Deepen preserves what's there.

Rewriting is the right call when the factual base itself is wrong or too vague to deepen. If the timeline is a vague narrative rather than timestamped events, or impact is unquantified, or the trigger is misidentified, deepen will compound the errors. Start over.
