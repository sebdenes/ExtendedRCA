# Example: third-party dependency timeout — `/rca deepen`

A shallow postmortem pinning a checkout degradation on "Stripe was slow," plus the deepened version.

## The scenario

The payment provider (Stripe, fictional) experiences a 12-minute latency spike during a busy shopping afternoon. The checkout service doesn't have a circuit breaker or a cached-fallback path, so every checkout waits for the full timeout before failing. The team's draft blames Stripe. The deepened version finds no circuit breaker, no vendor runbook, and an 8-minute delay in noticing the vendor's own status page.

This example exercises the **External dependencies** and **Code** fishbone categories and shows how the skill handles incidents where the trigger is legitimately external — the blameless-but-specific rigor still has plenty to find in the contributing and systemic layers.

## Files

- `shallow-draft.md` — "Stripe was slow" version.
- `deepened.md` — extended version with circuit-breaker and vendor-runbook findings.

## How to use

```
/rca deepen examples/third-party-timeout/shallow-draft.md
```

Differences worth noting: the shallow draft treats vendor failure as fully exogenous ("nothing we could do"). The deepened version agrees the trigger was the vendor but identifies our response weaknesses — no circuit breaker, no cached-price fallback, no one monitoring vendor status, manual communication with support — as the actual addressable surface. Corrective actions span all four taxonomy buckets and include a Respond action (vendor-status integration) that the shallow draft entirely missed.
