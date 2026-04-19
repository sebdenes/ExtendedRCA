# Example: cross-team breaking change — `/rca deepen`

A shallow postmortem blaming "the auth team" for breaking three downstream services, plus the deepened version.

## The scenario

The auth team deprecates a legacy token-validation endpoint they believe is unused, per their deprecation announcement in Slack two weeks prior. Three downstream services still depend on it. When the endpoint is removed, internal traffic fails across the three services for 37 minutes. The shallow draft has a blame-ful framing pointing at the auth team. The deepened version reframes the finding as a process gap: there is no deprecation policy, announcements in Slack aren't a blocking signal, and there are no contract tests between auth and downstream services.

This example exercises the **Process** and **People** fishbone categories and specifically shows how to keep the tone non-blameful while still being concrete about the gap.

## Files

- `shallow-draft.md` — the "auth team shouldn't have" version.
- `deepened.md` — the extended, blameless but specific version.

## How to use

```
/rca deepen examples/cross-team-breaking-change/shallow-draft.md
```

Differences worth noting: the shallow draft blames a team ("the auth team shouldn't have deprecated without warning") — a classic failure of RCA tone. The deepened version reframes this as a process gap: the auth team *did* announce the deprecation, twice, per existing norms; the norms themselves are insufficient. The systemic cause ("Slack announcements are not a blocking signal; there is no deprecation policy") is the actual fixable thing. The action set includes a deprecation policy (Prevent), contract tests (Prevent + Detect), a service-discovery registry (Mitigate), and a cross-team-change communication standard (Respond).
