# Fishbone Categorization for Software Incidents

The fishbone (Ishikawa) diagram is a tool for forcing comprehensive hypothesis generation. Classic manufacturing fishbones use the 6 M's (Manpower, Method, Machine, Material, Measurement, Milieu). Those don't map cleanly to software. This reference gives you six categories tuned for software incidents, with prompting questions for each.

The purpose is not to draw a literal fishbone diagram — it's to resist tunnel vision during hypothesis generation by asking "what could each of these categories have contributed?" for every incident, even when one category seems obviously responsible.

## The six categories

### People

Who was in the loop, what did they know, what context was missing?

Prompting questions:
- Was the person who made the triggering change familiar with the affected code path?
- Were reviewers familiar with the failure mode that occurred?
- Who else had relevant context that wasn't consulted? Why wasn't it?
- Was on-call well-matched to this incident? Did they have the runbooks and access they needed?
- Was there new-hire or recently-reorg'd ownership where knowledge transfer was incomplete?
- Were the right humans alerted at the right times, or did alerts go to a rotation that couldn't act?

Typical findings in this category: knowledge gaps from recent reorgs, reviewer routing that didn't surface cross-service risks, on-call without the context or access to act quickly.

### Process

How do changes get reviewed, deployed, tested, rolled back in general?

Prompting questions:
- What process should have caught this, and why didn't it?
- Was the change rushed, batched with unrelated changes, or deployed at an unusual time?
- Did the rollout process follow normal steps? If abbreviated, why?
- Does the review process require eyes on external-service calls / config changes / migrations?
- Is there a staging environment that represents production faithfully? If not, what's missing?
- How long does a rollback take, and why?
- Are there change-freezes or calendar constraints that mattered here?

Typical findings: review rules that don't route correctly, staging-production divergence, slow rollback mechanisms, deployment batching that hides risk.

### Code

What does the code actually do, and where did its behavior diverge from what was assumed?

Prompting questions:
- What was the specific defect, and what correct behavior would have prevented the impact?
- Was the code tested? If yes, why didn't the test catch it? If no, why not?
- Is this class of bug systematically untestable in the current setup?
- Were there guardrails (validation, type checks, assertions, circuit breakers) that should have caught this?
- Is the code path normally exercised, or was this edge-case behavior?
- Did a recent refactor change the behavior of shared utilities?

Typical findings: edge cases not exercised in tests, guardrails that don't exist or were disabled, shared utilities whose semantics changed without downstream updates.

### Infrastructure

Compute, storage, network, shared resources, quotas.

Prompting questions:
- Was any infrastructure at or near a limit (CPU, memory, connections, file descriptors, quotas)?
- Did any autoscaling, load-balancing, or routing behavior change?
- Were there recent infra changes (cluster upgrade, k8s version, load balancer config)?
- Did a noisy neighbor affect this workload?
- Was there a network partition, DNS change, or certificate expiry?
- Did the incident span regions? Did failover work as expected?

Typical findings: hitting undocumented limits, stale infra assumptions ("we'll never hit that quota"), failover paths that were never exercised.

### Data

Schema, volume, distribution, new patterns, poisoned inputs.

Prompting questions:
- Did data volume or velocity change recently?
- Was there a schema change, a new data type, or unusual input?
- Was there a specific input that triggered this (poison pill, pathological case)?
- Has the distribution shifted in a way that broke an assumption (seasonality, new cohort, new feature flag)?
- Were there ETL / migration steps that behaved differently under load or under this specific data?
- Did caching behave differently because of input patterns?

Typical findings: distribution shift breaking assumptions, new data patterns from upstream features, schema migrations with incomplete coverage.

### External dependencies

Upstream services, third-party APIs, providers.

Prompting questions:
- Did any upstream service change behavior — latency, response shape, error rates, rate limits?
- Were there any provider-side incidents happening at the same time?
- Did credential rotation, token expiry, or auth config affect the call path?
- Are there retry / timeout / circuit-breaker settings on this call path, and are they calibrated correctly?
- Did the external dependency announce a change you missed?

Typical findings: upstream behavior changes not noticed, missing or poorly-calibrated timeouts/retries, auth token expiries that cascade.

## How to use this in an analysis

1. After establishing the facts (Phase 1), spend a few minutes under each category asking the prompting questions above.
2. Generate at least one hypothesis per category where there's any plausible connection — even if weak. You can rule them out quickly.
3. For the top 3-4 hypotheses (across categories), continue into the 5-whys chain.
4. In the final artifact, list hypotheses considered and ruled out under each category briefly — this shows the analysis was comprehensive without bloating the doc.

## Avoiding category inflation

Six categories is a ceiling, not a floor. Some incidents genuinely have no data component or no external-dependency component. Don't invent content in an empty category just to fill it — write "no contributing factors identified" if that's honest. Empty categories are more trustworthy than padded ones.

## Example

Imagine a checkout outage. Without fishbone, you might land on "the deploy introduced a bug" and move on. With fishbone:

- **Code**: nil-handling bug in pricing client.
- **Process**: review didn't route to pricing-service owners.
- **People**: the engineer wasn't familiar with pricing-client nil semantics; reorg moved ownership last month.
- **Infrastructure**: rollback took 9 minutes because rollback requires manual approval.
- **Data**: no contributing factor identified.
- **External dependencies**: no contributing factor identified.

That's a richer picture. The actions flowing from it would include a review-routing fix (Process), a knowledge-transfer followup (People), *and* a rollback automation task (Infrastructure) — not just a code patch. The incident's impact duration was as much about rollback latency as it was about the deploy itself, and the narrow code-only view would have missed that entirely.
