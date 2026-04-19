# Incident: Feature store corruption — 2026-01-23

## What happened

We discovered that one of our key ML features (`user_lifetime_value_90d`) had been writing nulls since a schema migration on 2026-01-19. The ranking model quality dropped but nobody noticed until the weekly metrics review.

## Root cause

The migration script on 2026-01-19 dropped and recreated the column with the wrong default, so all subsequent writes inserted null.

## Timeline

- Jan 19 — migration ran
- Jan 23 — metrics review caught the drop in ranking quality
- Jan 23 — backfill kicked off
- Jan 24 — backfill complete

## Impact

Ranking quality was degraded for 4 days. Hard to quantify precisely but click-through was down ~3%.

## Action items

- Fix the migration script (done)
- Be more careful with migrations
