# Example: login outage — `/rca deepen`

A shallow postmortem draft, plus the deepened version you'd expect `/rca deepen` to produce.

## The scenario

A fictional login outage where the team wrote a one-paragraph postmortem pinning the incident on "a bad deploy" and scheduling a single ticket. This is the most common real-world RCA anti-pattern, and it's what `/rca deepen` exists to address.

## Files

- `shallow-draft.md` — the original postmortem, deliberately thin.
- `deepened.md` — the kind of output `/rca deepen shallow-draft.md` should produce.

## How to use this example

From inside Claude Code:

```
/rca deepen examples/login-outage/shallow-draft.md
```

Compare Claude's output against `deepened.md`. Differences worth noting:

- **Hypotheses considered and ruled out** appears in the deepened version. The shallow draft committed to "a bad deploy" without weighing alternatives (session-store issue, upstream auth provider, LB misconfiguration).
- **Contributing factors** are separated from the trigger. Shallow drafts tend to smoosh everything into "root cause."
- **Systemic causes** name a structural gap (review-routing rules don't require auth-team review for session-store changes). This is the layer that predicts whether the incident recurs.
- **Corrective actions are tagged** Prevent / Detect / Mitigate / Respond. The shallow draft's single ticket is all Prevent.

## Also useful for

Testing prompts, demonstrating the value of extended-rca to skeptical stakeholders, and training reviewers on what "deep enough" looks like.
