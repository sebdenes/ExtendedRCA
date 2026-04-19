# Postmortem — Silent feature store corruption (2026-01-19 → 2026-01-23)

## Summary

On 2026-01-19, a schema migration on the `feature_store` table silently began writing `NULL` for `user_lifetime_value_90d` in all new records. No alerts fired; no error paths were exercised. The ranking model consumed null-dominated feature vectors for four days before the degradation was caught during the weekly metrics review on 2026-01-23. Click-through rate on ranked content dropped ~3% during the window, with estimated revenue impact of $180k.

The deeper findings identify three contributing factors and one dominant systemic cause: ownership of feature-store data quality is unclaimed, with both the data-platform and ML teams assuming the other was responsible.

## Impact

- `user_lifetime_value_90d` populated with `NULL` for all records written between 2026-01-19 07:00 UTC and 2026-01-23 16:42 UTC (~4.4 days)
- Ranking model served ~340M scoring requests on degraded inputs
- Click-through rate dropped approximately 3.1% compared to the 28-day trailing average
- Estimated revenue impact: $180k (finance-confirmed range: $140k–$220k)
- No direct user harm beyond ranking quality; no data loss (historical rows unchanged)

## Timeline (UTC)

- Jan 16 — migration PR reviewed and approved by data-platform team
- Jan 19 07:00 — migration runs against production; completes "successfully"
- Jan 19 07:01 — first write to `user_lifetime_value_90d` inserts `NULL`; no alert
- Jan 19 08:14 — ranking model serves first degraded prediction; no alert
- Jan 20–22 — click-through rate drops gradually as cache of pre-migration features decays
- Jan 23 10:30 — weekly metrics review highlights CTR drop; investigation begins
- Jan 23 14:18 — feature vector inspection reveals null-dominated inputs
- Jan 23 15:03 — root cause traced to migration
- Jan 23 16:42 — fix deployed (default value restored)
- Jan 24 03:15 — backfill of affected rows complete

## Trigger

Schema migration applied at 2026-01-19 07:00 UTC.

## Proximate cause

The migration script dropped `user_lifetime_value_90d` and recreated it without a `DEFAULT` clause. The upstream writer (`feature-aggregator-service`) relies on the database default to populate this derived column when no explicit value is provided — which is the case for 94% of writes, because the value is computed in a downstream batch job and backfilled later. Post-migration, writes inserted `NULL` instead of the intended default of `0.0`.

## Contributing factors

1. **No pre-migration data-quality check.** The migration ran in production without first running against a staging copy and validating that column distributions matched expectations. The CI pipeline ran the migration but did not read any rows afterward.
2. **No shadow-read comparison during the 48 hours after migration.** Once the migration was in production, no automated job compared feature distributions before and after. A distribution-equality test would have caught the null-only writes within one hour.
3. **Detection relied entirely on a weekly human review.** There is no data-quality SLO on feature values — no alert on null rate, no alert on mean/median/p99 drift, no canary on model output quality. The 4-day detection lag is almost entirely attributable to this gap.

## Systemic / latent causes

1. **Feature-store data quality is unowned.** The data-platform team owns the storage layer and treats its job as keeping the tables available and the queries fast. The ML team owns model training and treats its job as iterating on models, assuming feature inputs are correct. The "correctness of derived feature values" lies between them, and neither team has it on their roadmap. A review of 2025 data-quality incidents shows the same gap has been a contributing factor in at least three of them. Until this ownership is made explicit (likely on the data-platform team's roadmap, with ML team sign-off on quality SLOs), this class of incident will recur.

## Hypotheses considered and ruled out

- **Upstream signal degradation.** Initial hypothesis was that user-activity signals (used to compute LTV) had degraded, perhaps from a tracking-pixel change. User-activity tables were unchanged; signal volume was on trend. Ruled out.
- **Model drift independent of features.** Could the model itself have degraded from input drift in a different feature? Feature-importance inspection showed `user_lifetime_value_90d` as the top-ranked input; nulls in this feature alone explained the observed CTR drop. Ruled out.
- **Cache poisoning of pre-computed features.** Investigated; the feature cache was regenerating correctly from storage, and storage was the source of nulls. The cache was propagating the problem faithfully. Ruled out.

## What went well

- Once the hypothesis converged on "something in feature-store changed recently," the migration log pointed at the root cause within 45 minutes.
- The backfill tooling was ready: a single command reconstructed the affected values from source tables in under 11 hours.
- Ranking model fell back gracefully on null inputs rather than crashing — the degradation was quality loss, not availability loss. Had it crashed, user-facing impact would have been worse.

## Corrective actions

**Prevent**
- [ ] Migration-pipeline requires a post-migration validation step: column distribution summary, flagged if any column shifts by >2σ from pre-migration. (Owner: data-platform; Due: 2026-02-20)
- [ ] Require explicit `DEFAULT` clauses in all migration scripts touching `feature_store.*`. Add lint rule. (Owner: data-platform; Due: 2026-02-10)
- [ ] Publish feature-store data-quality SLO with named owner. (Owner: VP of data + VP of ML, jointly; Due: 2026-02-28)

**Detect**
- [ ] Alert on null-rate per feature column (fires at 5× trailing average). (Owner: data-platform; Due: 2026-02-15)
- [ ] Shadow-read comparison job runs for 48 hours after every migration; alert on distribution deltas. (Owner: data-platform; Due: 2026-03-15)
- [ ] Model-output quality canary runs on every deploy and every upstream data change; alert on CTR deviation >1% vs. trailing. (Owner: ML platform; Due: 2026-03-30)

**Mitigate**
- [ ] Ranking model falls back to a simpler default-ranker when >10% of predictions have null top-importance features. (Owner: ML platform; Due: 2026-04-30)

**Respond**
- [ ] Add runbook for feature-store data incidents, including the distribution inspection query and the backfill command. (Owner: data-platform; Due: 2026-02-20)

## Open questions / unknowns

- Exact revenue impact. Finance's $140k–$220k range depends on counterfactual CTR assumptions; the uncertainty is unlikely to resolve without A/B-testing a reproduction, which isn't warranted.
- Whether any other derived features are missing explicit defaults. A one-time audit of `feature_store` schema is included as a Prevent action implicitly, but should be explicit — filed as follow-up.

## Lessons for the broader org

Data-quality ownership ambiguity between data-platform and ML has now been a contributing factor in at least three incidents in 2025 and the first one of 2026. Recommending a VP-level ownership decision, independent of the specific fixes above, as the highest-leverage intervention.
