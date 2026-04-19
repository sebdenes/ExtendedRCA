# Postmortem — Login outage (2026-03-14)

## Summary

On 2026-03-14 between 09:41 and 10:28 UTC, roughly 22% of login attempts failed with HTTP 500 errors. The incident was triggered by a deploy of `session-service v4.7` that shipped a serialization change incompatible with Redis-cached sessions from the previous version. It was resolved by rolling back to v4.6.

The deeper findings below identify three contributing factors and two systemic causes that made this incident possible and slow to detect.

## Impact

- ~22% of login attempts failed across a 47-minute window
- Two enterprise customers filed support tickets; one CSM escalated to VP
- Mobile app users were disproportionately affected (older session tokens)
- No data loss; no persistent account impact

## Timeline (UTC)

- 09:35 — `session-service v4.7` deployed to production (staged rollout, 10% → 50% → 100% over 4 min)
- 09:41 — first elevated error rate on `/login`; no alert fires (threshold 5%, observed peak 22% but averaging 11% over 5-min window)
- 09:48 — customer support ticket escalates; SRE on-call paged manually
- 09:54 — on-call identifies error rate, begins investigation
- 10:05 — correlation with deploy identified
- 10:18 — rollback initiated
- 10:28 — rollback complete; error rate returns to baseline

## Trigger

Deploy of `session-service v4.7` at 09:35 UTC.

## Proximate cause

`session-service v4.7` changed the Redis-cached session token format from JSON to MessagePack without a compatibility shim. Sessions cached by v4.6 (still active in Redis) deserialized to garbage under v4.7, causing panic-and-retry loops in the login path.

## Contributing factors

1. **Alert threshold was too coarse.** The error-rate alert fires on a 5% threshold over a 5-minute rolling average. The incident peaked at 22% instantaneous but averaged 11%, still above threshold — but the alert had a 9-minute evaluation delay (configured to reduce false positives), so detection lagged 7 minutes behind the first failed request.
2. **Rollback required manual approval.** Rollbacks are gated on a change-management workflow that added ~5 minutes of human latency. During active user impact, this gate is almost always approved immediately — the gate adds latency without preventing anything.
3. **Staged rollout did not include a canary gate.** The 10/50/100 staged rollout proceeded on a time schedule, not on error-rate telemetry. By the time the incident was visible, the deploy was already at 100%.

## Systemic / latent causes

1. **Cross-service change review doesn't account for shared infrastructure.** The session-service PR was reviewed by two session-service engineers, neither of whom is on the auth team or familiar with how session tokens are consumed across services. The format change was a cross-service breaking change disguised as an internal refactor. Review-routing rules don't require auth-team review for session-format changes. The same pattern — a team shipping a change to shared state without review by consumers — shows up in the last four cross-team incidents.
2. **Schema-compatibility testing is not part of the deploy pipeline.** There is no automated check that new service versions can coexist with cached state from previous versions during a rollout. This gap was flagged in an October 2025 design review but deprioritized in favor of feature work.

## Hypotheses considered and ruled out

- **Upstream auth provider (Okta) outage.** Okta status page showed green throughout the incident window; Okta-callback metrics showed no elevated latency.
- **Redis infrastructure issue.** Redis node health metrics (CPU, memory, connection count) were nominal. The issue was at the application layer (serialization), not infrastructure.
- **Load-balancer misconfiguration.** The LB routed traffic correctly to both v4.6 and v4.7 pods during the rollout; the errors came from v4.7 pods, not from LB misrouting.

## What went well

- Rollback, once approved, completed in 10 minutes — the rollback tooling itself is solid.
- The on-call engineer correlated with the deploy within 11 minutes of starting to look, which is fast for a non-obvious cause.
- Customer communication went out within 25 minutes of detection.

## Corrective actions

**Prevent**
- [ ] Add schema-compatibility test to CI: new service version must successfully deserialize sessions written by the previous version (Owner: session-service team; Due: 2026-04-30)
- [ ] Update review-routing: changes to session token format require auth-team review (Owner: platform team; Due: 2026-04-15)

**Detect**
- [ ] Reduce error-rate alert evaluation window from 5 min to 1 min for critical login path (Owner: SRE; Due: 2026-04-07)
- [ ] Add deploy-correlated error-rate alert that fires if error rate rises within 15 min of a deploy (Owner: SRE; Due: 2026-04-20)

**Mitigate**
- [ ] Add canary-gate to staged rollout: 10% stage holds until error rate is within 0.5× of baseline for 3 min (Owner: platform team; Due: 2026-05-15)

**Respond**
- [ ] Remove manual-approval gate from rollbacks during active P1 incident (keep it for non-incident rollbacks) (Owner: change-management working group; Due: 2026-04-30)

## Open questions / unknowns

- How many sessions cached by v4.6 were still being read at 09:41? We don't have per-session age telemetry, so we can't estimate the "blast" precisely. Consider adding session-age histogram metrics.
- Were any of the failed logins retried successfully after rollback? The login retry UX varies between web and mobile clients; worth a follow-up review.
