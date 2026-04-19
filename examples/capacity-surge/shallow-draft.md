# Incident: Checkout outage during flash sale — 2026-02-11

## What happened

During our Black Friday flash sale, checkout started timing out for a lot of users. We had to scale up the database connection pool and things recovered.

## Root cause

The connection pool was too small for the traffic we got.

## Timeline

- 11:00 — flash sale started
- 11:04 — checkout errors started
- 11:18 — we realized the pool was saturated
- 11:26 — pool size doubled, errors dropped
- 11:30 — back to normal

## Impact

Checkout was broken for ~25 minutes during peak sale traffic. We probably lost a chunk of revenue.

## Action items

- Double the connection pool size permanently
- Make sure this doesn't happen again
