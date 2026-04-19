# Postmortem — Checkout outage during flash sale (2026-02-11)

## Summary

On 2026-02-11 between 11:04 and 11:30 UTC, the checkout path returned HTTP 503 for approximately 38% of requests during a Black Friday flash sale. Traffic at peak was 14× the 28-day trailing average. The incident was triggered by the sale launch; the proximate cause was connection-pool exhaustion on `checkout-db`. It was mitigated by doubling the pool size at 11:26.

The deeper findings identify three contributing factors (no surge load test, no queue-depth alert, pool shared with lower-priority writes) and one dominant systemic cause (capacity planning has no accountable owner and has been deferred for six consecutive quarters).

## Impact

- Approximately 38% of checkout attempts between 11:04 and 11:30 returned 503 errors (26 minutes)
- Estimated $420k in abandoned carts (gross merchandise value), of which approximately 30% recovered via retry over the next hour
- 11 support tickets from customers, 2 escalations to senior leadership
- No data corruption; no payment double-charges

## Timeline (UTC)

- 10:55 — marketing email campaign deploys
- 11:00 — flash sale starts; traffic ramps from ~300 rps to ~4,200 rps over 90 seconds
- 11:04 — `checkout-service` latency p99 exceeds 2s; first 503s observed
- 11:07 — CPU alert fires on `checkout-service` (incorrectly — actual bottleneck was downstream)
- 11:10 — on-call begins investigating CPU; spends ~6 minutes on the wrong hypothesis
- 11:16 — database team paged after error pattern reviewed
- 11:18 — pool-exhaustion metric surfaced on `checkout-db` dashboard
- 11:22 — pool size increase drafted
- 11:26 — pool size doubled from 200 to 400; error rate drops within 30s
- 11:30 — error rate below 1%; incident closed

## Trigger

Flash-sale traffic spike (14× baseline) starting at 11:00 UTC.

## Proximate cause

The `checkout-db` connection pool was sized at 200 connections, with a soft limit of 180 to leave headroom. Under sustained 4,200 rps on checkout, held connections exceeded 180 within 4 minutes, queue depth grew unbounded, and new requests began timing out after the 2s wait budget.

## Contributing factors

1. **No surge-traffic load test before the sale.** The sale was planned three weeks in advance. Load tests existed for 2× and 5× baseline but not 14×. The 14× number came from the marketing team's internal projection and was never shared with engineering.
2. **No queue-depth or pool-utilization alert.** Alerting covered CPU, memory, and error rate on `checkout-service`, but not pool saturation on `checkout-db`. This caused the on-call to spend 6 minutes investigating CPU before reaching the actual bottleneck.
3. **The pool is shared with lower-priority writes.** Batch writes for analytics share the `checkout-db` pool with user-facing checkout queries. During the surge, an analytics batch job held ~40 connections for 2–3 minutes, crowding out checkout queries at the worst moment. This has been filed as tech debt since September 2025.

## Systemic / latent causes

1. **Capacity planning has no accountable owner.** Capacity review was nominally owned by "the platform team" but was deprioritized in every quarterly planning session for the last six quarters. The decision memo for 2025-Q4 explicitly noted "defer capacity work until next quarter." The same pattern — proximate capacity failures, acknowledged tech debt, perpetual deferral — shows up in the April 2025 and November 2025 incidents. Until capacity planning is owned by a named individual with a roadmap slot, this class of incident will recur.
2. **Marketing and engineering have no shared forecast surface.** The 14× traffic projection existed in a marketing spreadsheet for three weeks. There is no forum in which marketing forecasts are joined to engineering capacity plans. This is an organizational gap, not a tooling gap.

## Hypotheses considered and ruled out

- **CDN / edge cache issue.** The on-call initially suspected a cache miss storm. Edge-cache hit rate was 94% throughout the incident, consistent with the 7-day average. Ruled out.
- **Memory pressure on `checkout-service`.** Heap utilization was 62%, well below the 80% alert threshold. Ruled out.
- **Code bug introduced by recent deploy.** No deploys in the 48 hours before the sale. Ruled out.

## What went well

- Once the correct hypothesis was identified, mitigation was fast: the pool-size change took 4 minutes from first edit to full effect.
- Rollback of the analytics batch job (which had been automatically triggered by the sale's data volume) was clean and didn't require manual intervention.
- Customer-facing status page was updated within 9 minutes of incident confirmation.

## Corrective actions

**Prevent**
- [ ] Run a 15× load test before every announced sale. Block the sale launch until the test passes. (Owner: platform team; Due: 2026-03-15)
- [ ] Separate analytics-batch pool from user-facing checkout pool. (Owner: data platform; Due: 2026-04-30)
- [ ] Establish a named owner for capacity planning, with quarterly roadmap commitment. (Owner: VP of engineering; Due: 2026-03-01)

**Detect**
- [ ] Add pool-utilization and queue-depth alerts on `checkout-db` (page at 70%, critical at 85%). (Owner: SRE; Due: 2026-02-28)
- [ ] Add deploy-independent surge-detection alert (error rate + traffic ramp). (Owner: SRE; Due: 2026-03-15)

**Mitigate**
- [ ] Implement dynamic pool-size scaling tied to upstream request rate. (Owner: platform team; Due: 2026-05-01)

**Respond**
- [ ] Establish a joint marketing-engineering forecast review one week before any announced sale. (Owner: VP of engineering + VP of marketing; Due: next sale cycle)
- [ ] Add a runbook for pool-exhaustion incidents, including the analytics-batch kill command. (Owner: SRE; Due: 2026-02-28)

## Open questions / unknowns

- Exact GMV lost. The $420k estimate assumes cart-abandonment conversion at our 7-day average; retry-recovered orders are easier to measure but total loss is harder. Finance will publish a reconciled figure in the Q1 review.
- Whether the analytics-batch contention alone would have triggered a smaller-scale incident at lower surge levels. We don't have data at intermediate traffic multipliers.

## Lessons for the broader org

This incident, along with the April and November 2025 incidents, suggests capacity planning has been systematically under-resourced for at least 18 months. Recommending an org-wide capacity-ownership decision as a higher-priority investment than any of the specific actions above.
