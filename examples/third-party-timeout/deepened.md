# Postmortem — Payment provider latency spike (2025-12-10)

## Summary

On 2025-12-10 between 14:58 and 15:22 UTC, the third-party payment provider (`stripe-api.com`) experienced elevated p99 latency, rising from the usual ~180ms to a sustained 8–14s with occasional 20s+ responses. Our checkout service had no circuit breaker and no cached-price fallback, so every checkout attempt waited the full 15s client timeout before failing, cascading load into adjacent services. Incident duration end-to-end: 24 minutes.

The trigger was legitimately external. The deeper findings are about our response surface: no circuit breaker, no fallback, an 8-minute lag in noticing the vendor's own status page, and no escalation path to vendor support.

## Impact

- Approximately 64% of checkout attempts between 14:58 and 15:22 either timed out or were abandoned
- Elevated latency on adjacent services (cart, user-profile) due to cascading wait times
- Estimated $85k in abandoned carts (of which ~60% recovered via customer retry)
- 23 support tickets, 1 social-media complaint thread

## Timeline (UTC)

- 14:55 — Stripe begins experiencing degradation (confirmed post-hoc from Stripe's own incident report)
- 14:58 — first elevated checkout latency observed internally
- 15:02 — on-call paged; initial hypothesis is internal
- 15:06 — Stripe status page shows incident (we don't notice for 8 more minutes)
- 15:09 — on-call rules out internal causes; begins looking at dependencies
- 15:14 — on-call manually checks Stripe status page; confirms vendor issue
- 15:16 — customer-facing status page updated
- 15:18 — Stripe recovery begins
- 15:22 — checkout latency returns to baseline

## Trigger

Upstream latency degradation on `stripe-api.com`.

## Proximate cause

The checkout service's Stripe client has a 15-second total timeout and no circuit breaker. Under sustained vendor latency of 8–14s, every request consumed connection-pool slots for the full timeout. The pool of 250 connections saturated within 3 minutes, and new requests queued indefinitely. There is no cached-price fallback path: checkout is hard-gated on a live price-confirmation call to Stripe, even though price is known to the cent from the cart step.

## Contributing factors

1. **No vendor-status monitoring.** Stripe's status page was updated at 15:06, but we had no integration polling it. The 8-minute detection lag between Stripe's own acknowledgment and our on-call noticing is pure observability gap.
2. **No circuit breaker on the Stripe client.** A circuit breaker with 50% error-rate threshold and 5-second open duration would have shed load within 30 seconds of the upstream degradation, preserving the connection pool and allowing cached-fallback paths to serve other requests.
3. **Hard dependency on live Stripe call for price confirmation.** The architecture sends every checkout through a synchronous Stripe call even when the price is known from cart. A cached-price fallback with async reconciliation would have let 80%+ of checkouts succeed during the vendor incident.
4. **No escalation path to vendor support.** We have a Stripe account-manager contact in a wiki page no one can find under incident conditions. During the window, no one attempted to reach vendor support for confirmation or ETA.

## Systemic / latent causes

1. **Vendor resilience is treated as vendor responsibility.** The organizational pattern is "if it's vendor-side, we wait." This pattern extends to architecture (no fallbacks), observability (no vendor-status integration), and process (no vendor-escalation runbook). It presumes that vendor outages are brief and total, when in practice partial degradation is common and extended. The same pattern is visible in the October 2025 mapping-provider incident.
2. **Checkout architecture treats external calls as reliable.** There is no `references/degradation-modes.md` or equivalent documenting what checkout does when any given external dependency is slow vs. failed. Design reviews don't explicitly evaluate failure-mode behavior. This is a design-culture gap, not a specific code gap.

## Hypotheses considered and ruled out

- **Internal network issue.** First hypothesis during initial triage. Cross-region latency metrics were nominal; ruled out within 7 minutes.
- **Deploy-induced issue on our side.** No deploys in the 12 hours before the incident. Ruled out quickly.
- **DDoS or traffic spike.** Traffic was at or below baseline for the hour. Ruled out.

## What went well

- Once the on-call checked Stripe's status page, hypothesis convergence was fast (under 2 minutes).
- Customer-facing status page was updated within 2 minutes of vendor-issue confirmation.
- No data loss, no double-charges, no persistent state issues — the timeout failure mode was clean.

## Corrective actions

**Prevent**
- [ ] Implement circuit breaker on Stripe client (50% error-rate threshold, 5s open duration, half-open probe). (Owner: payments team; Due: 2026-01-15)
- [ ] Implement cached-price checkout path with async Stripe reconciliation. Design doc first, then phased rollout. (Owner: checkout team; Due: 2026-03-31)

**Detect**
- [ ] Integrate Stripe status-page RSS feed with internal alerting. Page on any "major" or "investigating" status within 60 seconds. (Owner: SRE; Due: 2026-01-10)
- [ ] Add external-dependency latency dashboard covering Stripe, mapping provider, and email provider. (Owner: SRE; Due: 2026-01-20)
- [ ] Alert on Stripe p99 latency >2× trailing 7-day median. (Owner: SRE; Due: 2026-01-10)

**Mitigate**
- [ ] Document degradation-mode behavior for every external dependency used by checkout. (Owner: checkout team; Due: 2026-02-28)

**Respond**
- [ ] Publish vendor-escalation runbook with Stripe account-manager phone number and Slack channel. Test in next tabletop exercise. (Owner: payments team + SRE; Due: 2026-01-15)
- [ ] Add external-dependency section to every future postmortem template. Force explicit reasoning about vendor-response weaknesses even when vendor was the trigger. (Owner: incident-response working group; Due: 2026-02-01)

## Open questions / unknowns

- Did the cascading load into cart and user-profile services cause any measurable degradation for users not attempting checkout? Telemetry suggests small effect but we don't have counterfactual data.
- Would a 5-second client timeout (vs. current 15s) have been better? Shorter timeout means faster failure and shed load, but also more user-visible failures during transient glitches. Trade-off is worth a design discussion.

## Lessons for the broader org

The "vendor problem = not our problem" pattern has now cost measurable revenue twice in three months. Recommending a resilience-review pass across all critical external dependencies (payments, mapping, email, SMS) independent of the specific fixes above.
