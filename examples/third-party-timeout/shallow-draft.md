# Incident: Payment provider slowdown — 2025-12-10

## What happened

Stripe had a latency spike in the afternoon. Our checkout got slow and then started timing out. We couldn't do anything about it, it was on Stripe's side.

## Root cause

Stripe was slow.

## Timeline

- ~3pm — checkout started getting slow
- ~3:10pm — we confirmed it was Stripe
- ~3:20pm — Stripe posted on status page
- ~3:25pm — Stripe recovered

## Impact

Checkout was degraded for about 25 minutes.

## Action items

- Nothing to do, Stripe's problem
