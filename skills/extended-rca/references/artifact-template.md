# Written Artifact Template

This is the canonical structure for the RCA document. Each section has a specific purpose — the ordering and separation are deliberate. Follow the structure; fill in with content from the analysis phases.

## Template

```markdown
# [Incident name — short, descriptive, e.g. "Checkout outage — 2026-04-19"]

## Summary

<3–5 sentences in plain language. What happened, when, who was affected, how it was resolved. Written for someone who wasn't in the room — a peer engineer on a different team, a new hire next quarter. No jargon without explanation. Don't be clever; be clear.>

## Impact

<Quantified where possible. Good: "17% of checkout requests failed between 14:02 and 14:47 UTC, ~2,300 users affected, estimated $4,100 in dropped order revenue. SLO burned 7% of monthly error budget." Bad: "some users couldn't check out for a while.">

- **Duration:** <start timestamp> — <end timestamp> (<total minutes>)
- **Affected services:** <list>
- **Affected users:** <count / segment>
- **Estimated business impact:** <dollars / orders / etc. where known; "unknown" is acceptable>
- **SLO impact:** <error budget burn / SLA clock>

## Timeline

<Tight chronology. One line per significant event, timestamped in UTC. Include at minimum: first alert, escalation, diagnosis milestones, mitigation, full resolution. Don't narrate; list.>

- **14:02 UTC** — Deploy X.Y.Z merged and began rollout
- **14:06 UTC** — Checkout error rate alert fired (9% above baseline)
- **14:11 UTC** — On-call engineer acknowledged and began investigation
- **14:23 UTC** — Hypothesis: new pricing-service client code path
- **14:31 UTC** — Rollback initiated (required manual approval)
- **14:47 UTC** — Rollback completed; error rate returned to baseline
- **15:05 UTC** — Incident declared resolved

## Root cause analysis

### Trigger

<The immediate event that made the incident visible. One or two sentences. Don't editorialize here.>

### Proximate cause

<The mechanism by which the trigger produced the impact. Be specific about the code path, config, or resource. One paragraph.>

### Contributing factors

<Conditions that amplified or enabled the proximate cause but weren't sufficient on their own. Bullet list. Each item should be a standalone factor, not a restatement of the proximate cause.>

- <Factor 1, specific and named>
- <Factor 2, specific and named>
- <Factor 3, specific and named>

### Systemic / latent causes

<The deeper organizational, process, or design conditions that allowed the contributing factors to exist. This is where an extended RCA distinguishes itself from a shallow one.>

<Use a why-chain (see references/five-whys.md) if it clarifies. Example shape:>

**Why was the pricing-client nil-handling defect possible?**

The pricing-service client didn't handle nil responses
→ because the code path was added under time pressure with no nil-response test
→ because the pricing-service client's tests don't fuzz upstream-response shape
→ because there's no policy for nil-response handling on external-service calls and no owner for the pricing-service client after the Q1 refactor
→ because service ownership boundaries between payments and pricing teams became ambiguous after the January reorg and haven't been revisited.

<If there are multiple converging chains, show each one separately. Don't collapse them into a single narrative.>

## Hypotheses considered and ruled out

<Short list. For each: the hypothesis, and the specific evidence that ruled it out. This protects future analyses from re-exploring the same dead ends and demonstrates the current analysis was rigorous.>

- **Upstream pricing-service degradation**: pricing-service p99 latency and error rates were normal throughout the window.
- **Database connection exhaustion**: connection pool metrics showed <30% utilization throughout.
- **Traffic spike**: request volume was within 1σ of seasonal baseline.

## What went well

<Specific things about detection and response that should be preserved. Being explicit about these protects good patterns from being eroded by future process changes.>

- Alert fired and on-call acknowledged within 9 minutes of first error
- Hypothesis narrowed correctly within 12 minutes of acknowledgment
- Rollback tooling, once approval was granted, completed in 16 minutes
- Customer communication went out within 20 minutes of acknowledgment

## Corrective actions

<Table or structured list. Each action tagged Prevent / Detect / Mitigate / Respond. Include owner (or TBD) and priority.>

| Action | Type | Owner | Priority | Addresses |
|---|---|---|---|---|
| Add contract test for nil responses from pricing service | Prevent | payments team | High | Proximate cause |
| Define nil-response policy for all external-service calls | Prevent | eng leadership | High | Systemic |
| Route reviews involving pricing-client changes to pricing-service owners | Prevent | platform team | Medium | Contributing factor |
| Add SLO alert on pricing-service error rate (not just checkout) | Detect | payments team | Medium | Contributing factor |
| Make pricing-service client fail open to cached default on nil | Mitigate | payments team | Medium | Systemic |
| Remove manual-approval step from rollback flow for payments services | Respond | platform team | High | Contributing factor |
| Revisit service ownership map post-reorg | Prevent | eng leadership | High | Systemic |

<Aim for content in at least three of Prevent/Detect/Mitigate/Respond. A list of only Prevent actions implicitly assumes the team will never fail again — that's not a safe assumption.>

## Open questions / unknowns

<Things the evidence can't answer. Being honest here is more useful than guessing.>

- Why did the alerting threshold not trip until 4 minutes into the incident? Threshold calibration wasn't investigated in this analysis.
- How many of the ~2,300 affected users retried successfully after the window? Retry data wasn't pulled.

## Lessons for the broader org

<Optional. Use only when systemic causes point at something larger than this one incident. Keep short — one or two paragraphs at most.>

<Example: "This incident, along with [ref to previous two incidents], suggests service ownership ambiguity post-January-reorg is a recurring contributing factor. Recommending an org-wide ownership-map review as a higher-priority investment than any of the specific actions above, on the grounds that fixing the ownership map would address multiple classes of incident.">
```

## Notes on using the template

- **Don't skip sections.** If a section has no content, write "None identified" — don't delete the heading. Empty sections tell the reader you considered and ruled out that layer of cause.
- **Be specific.** Service names, endpoints, config keys, commit hashes, flag names. Vague postmortems don't generate vague learning — they generate no learning.
- **Attribute to systems, not people.** See the Tone section of SKILL.md.
- **If you skipped a phase** (e.g. you didn't generate competing hypotheses because the proximate cause was clear from the first read of the logs), note that explicitly in the artifact — either in the "hypotheses considered" section as "only one causal narrative was considered because X" or in "open questions." Transparency about method is part of a rigorous RCA.
