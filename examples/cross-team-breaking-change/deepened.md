# Postmortem — Cross-team breaking change from auth deprecation (2026-03-28)

## Summary

On 2026-03-28 between 10:00 and 10:37 UTC, the `/validate-token-v1` endpoint on the auth service was removed, breaking three downstream services (`search`, `notifications`, `admin-panel`) that still depended on it. The deprecation had been announced in `#eng-announcements` on 2026-03-13 and again on 2026-03-24, per the current organizational norm. The trigger was the endpoint removal; the proximate cause was the three downstream services still consuming it. The deeper findings are about why the existing deprecation norm doesn't work.

Note: this incident is not the fault of the auth team. They followed the existing deprecation norm. The norm itself is insufficient.

## Impact

- `search` service: 100% of internal search requests failed for 37 minutes; user-facing search fell back to a degraded keyword-only mode (still usable, lower quality)
- `notifications` service: ~88% of notification deliveries queued (eventually delivered after recovery)
- `admin-panel`: internal tool unusable for 37 minutes (no external impact)
- Customer-facing: ~14% of search sessions experienced degraded results during the window; no direct revenue impact

## Timeline (UTC)

- 2026-03-13 11:24 — auth team posts deprecation notice in `#eng-announcements`, 2-week timeline
- 2026-03-24 09:00 — auth team posts reminder in `#eng-announcements`
- 2026-03-28 10:00 — `/validate-token-v1` removed from production
- 10:02 — `search` service errors begin; alert fires
- 10:04 — `notifications` service errors begin
- 10:06 — `admin-panel` errors begin
- 10:08 — on-call for search paged
- 10:11 — on-call for notifications paged (separate on-call rotation)
- 10:13 — cross-team huddle convened in incident channel
- 10:15 — correlation to auth change identified via git-log search
- 10:22 — decision: restore endpoint temporarily, give downstream teams 30 days to migrate
- 10:30 — auth team restores endpoint (revert merged)
- 10:37 — three services recover

## Trigger

Removal of `/validate-token-v1` at 10:00 UTC.

## Proximate cause

Three downstream services still depended on `/validate-token-v1` at the time of removal. They had not been aware of the deprecation because:
- `search` team: the on-call who saw the Slack announcement on 2026-03-13 was traveling the week of 2026-03-24 and didn't see the reminder; no one else on the team noticed.
- `notifications` team: the service's on-call rotation doesn't monitor `#eng-announcements` as part of its duties; the channel has 30+ messages/day.
- `admin-panel`: the service is officially owned by "the platform team" but has no active maintainer; no one is reading announcements on its behalf.

## Contributing factors

1. **Slack announcements are not a blocking signal.** The current deprecation norm is to post in `#eng-announcements` and wait. There is no acknowledgment mechanism, no escalation on silence, no automated check that announced deprecations have been noticed by consumers.
2. **No contract tests between auth and its consumers.** A contract test that exercised `/validate-token-v1` would have failed in CI when the endpoint was removed, catching the breaking change before deploy.
3. **No service-to-consumer registry.** The auth team had no automated way to identify which services were calling `/validate-token-v1`. They estimated consumers from informal memory (which was wrong by three services).
4. **Orphaned service ownership.** `admin-panel` has no active maintainer. Its on-call rotation points at a team that doesn't consider it theirs.

## Systemic / latent causes

1. **The organization has no deprecation policy.** The current norm ("announce in Slack, wait two weeks, remove") is an informal convention. It specifies no SLA for consumer acknowledgment, no escalation if consumers don't respond, no requirement for dependency analysis before announcement, and no rollback procedure if consumers turn out to still depend. Every cross-team breaking change inherits this ambiguity. The same pattern — producer team followed informal norms, consumer team missed the announcement, breakage followed — has now appeared in this incident and in the October 2025 pricing-client-v2 incident.
2. **Service ownership metadata is not enforced.** The organization has a service registry, but ownership is self-reported and not required. Orphaned services like `admin-panel` pass through every planning review unnoticed.

## Hypotheses considered and ruled out

- **A misconfiguration on the auth service.** The removal was clean; the service was healthy; the 404 responses were expected behavior for the removed endpoint. Ruled out.
- **A DNS or service-discovery issue.** Network paths between services were healthy; the failures were HTTP 404s from auth itself, not connection errors. Ruled out.
- **The three downstream services sharing a common library with a stale client.** Investigated; the three services have independent codebases and made the call directly. No shared client library was involved.

## What went well

- Detection fired within 2 minutes of the first broken request — alerting on search and notifications is well-tuned.
- Cross-team coordination in the incident channel was fast: four teams in the same room within 13 minutes.
- The auth team's revert was clean — a single PR revert, no database or config complications.
- `search` service's fallback to keyword-only mode preserved a usable experience for external users. That degradation path should be preserved and replicated.

## Corrective actions

**Prevent**
- [ ] Publish an organizational deprecation policy (mandatory consumer acknowledgment, 30-day minimum, dependency analysis required before announcement, named escalation path). (Owner: VP of engineering; Due: 2026-05-15)
- [ ] Require contract tests between any producer/consumer service pair. Starting with auth and its top 10 consumers. (Owner: platform team; Due: 2026-06-30)
- [ ] Enforce ownership metadata in the service registry — services without an owning team block CI. (Owner: platform team; Due: 2026-04-30)

**Detect**
- [ ] Automated dependency-analysis tool: given a producer service, list all consumers (via logs, tracing, or service-mesh data) at the time of an announced deprecation. (Owner: platform team; Due: 2026-06-01)
- [ ] Contract-test failure blocks merge on both producer and consumer side. (Owner: CI team; Due: 2026-06-30)

**Mitigate**
- [ ] Service-discovery registry with canonical ownership, contact channels, and consumer list. (Owner: platform team; Due: 2026-07-31)
- [ ] Reactivate `admin-panel` ownership: assign a named team or decommission the service. (Owner: VP of engineering; Due: 2026-04-30)

**Respond**
- [ ] Cross-team breaking-change runbook: named on-call escalation for producer and consumer teams, standard rollback-first playbook. (Owner: incident-response working group; Due: 2026-05-15)
- [ ] Add a "deprecation announcements" email digest — opt-out rather than opt-in, delivered weekly. Fallback channel for people who miss Slack. (Owner: DevEx; Due: 2026-04-30)

## Open questions / unknowns

- How many other deprecations are currently in-flight under the informal norm? A one-time audit would be worthwhile.
- Is `admin-panel` actually used? If not, decommissioning is the right call. If it is, who should own it? Worth a follow-up ownership-review thread.

## Lessons for the broader org

Two cross-team-breaking-change incidents in six months suggest the informal Slack-announcement norm has structurally outlived its usefulness as the organization has grown. A formal deprecation policy is the single highest-leverage investment from this incident, separate from the specific tooling actions above.
