# Incident: Auth deprecation broke downstream services — 2026-03-28

## What happened

The auth team deleted the old `/validate-token-v1` endpoint. Three services (search, notifications, admin-panel) were still calling it. Traffic failed for 37 minutes until the endpoint was temporarily restored.

## Root cause

The auth team deprecated the endpoint without warning the other teams.

## Timeline

- 10:00 — auth team removed `/validate-token-v1`
- 10:02 — search service errors start
- 10:08 — on-call paged
- 10:15 — correlation to auth change identified
- 10:30 — auth team restored the endpoint temporarily
- 10:37 — services recover

## Impact

Three internal services were broken for 37 minutes. Some external users saw errors on search.

## Action items

- Auth team should warn people before deprecating stuff
