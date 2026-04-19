# Example: silent data corruption — `/rca deepen`

A shallow postmortem blaming "a bad migration script" for ML feature-store corruption, plus the deepened version.

## The scenario

A schema migration on the ML feature store writes nulls for a derived feature that previously had meaningful values. No alerts fire — the migration "succeeded." Downstream model quality degrades for four days before being noticed. The shallow postmortem says the migration script had a bug; the deepened version finds no shadow-read validation, no data quality SLO, and unclear ownership between the data-platform and ML teams.

This example exercises the **Data** and **People** fishbone categories and shows how an RCA benefits from rigorous layer separation when the trigger and proximate cause are far apart in time.

## Files

- `shallow-draft.md` — "the migration had a bug" version.
- `deepened.md` — the extended version with systemic ownership findings.

## How to use

```
/rca deepen examples/silent-data-corruption/shallow-draft.md
```

Differences worth noting: the shallow draft terminates at "the migration had a bug," a textbook single-cause narrative. The deepened version distinguishes the trigger (the migration run) from the proximate cause (default-null backfill on a derived column) from the contributing factors (no pre-migration data-quality check, no shadow-read comparison, 4-day detection lag) from the systemic cause (unclear ownership — neither the data-platform team nor the ML team considered feature-store integrity their responsibility). The action set spans all four taxonomy buckets.
