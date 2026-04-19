# Example: capacity surge — `/rca deepen`

A shallow postmortem attributing a flash-sale outage to "needing a bigger connection pool," plus the deepened version.

## The scenario

A retail site runs a flash sale that drives 14× normal traffic for 22 minutes. The checkout service exhausts its database connection pool and starts timing out. The team's instinctive postmortem pins it on pool size. The deepened version finds no capacity-planning process, no queue-depth SLO, and an incentive structure that has consistently lost to feature work.

This example exercises the **Infrastructure** and **Process** fishbone categories and the classic trap of treating capacity as a knob problem rather than a process problem.

## Files

- `shallow-draft.md` — the "increase the pool size" postmortem.
- `deepened.md` — the extended version with contributing and systemic layers.

## How to use

```
/rca deepen examples/capacity-surge/shallow-draft.md
```

Differences worth noting: the shallow draft has one action (enlarge the pool), all Prevent. The deepened version separates the trigger (Black Friday sale) from the proximate cause (pool exhaustion under sustained contention) from the contributing factors (no surge-traffic load test, no queue-depth alert, shared pool with lower-priority writes) from the systemic cause (capacity planning has no owner and has been deferred for six consecutive quarters). It adds Detect, Mitigate, and Respond actions.
