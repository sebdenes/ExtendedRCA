# Trends synthesis — `/rca trends <folder>`

In trends mode, you ingest N postmortems from a folder and synthesize recurring systemic themes across them. This is the payoff that disciplined per-incident layer-separation enables: once you've tagged every contributing factor and systemic cause consistently, you can see which ones show up repeatedly — and those recurring items are where org-level investment actually moves the needle.

A single postmortem tells you what went wrong. A trend report tells you *what keeps going wrong*.

## When this mode is useful

- Quarterly or annual postmortem review. You have 10–30 postmortems from the last window and want to identify where to spend engineering capital.
- After a spike in a specific class of incident (latency, capacity, data integrity). You want to know whether the spike reflects an underlying pattern or is coincidence.
- Before an org-level investment decision (a new reliability team, an SRE handbook, a deprecation policy). Trends give you evidence for the proposal.

This mode is explicitly NOT useful for a single incident or a pair of incidents. The synthesis needs enough data points to distinguish signal from noise — rule of thumb, at least 5 postmortems for weak trends and 10+ for strong ones.

## Workflow

### 1. Ingest

Read every file in the folder that looks like a postmortem (usually `*.md`). For each:

- Extract the trigger, proximate cause, contributing factors, and systemic / latent causes — using whatever layer-separation the postmortem has. If a postmortem doesn't separate layers cleanly, do your best to infer them, and note this as a data-quality caveat in the output.
- Extract the corrective actions and their tags (Prevent / Detect / Mitigate / Respond).
- Capture the date, duration, and impact if available.

If any postmortem is too shallow to extract these cleanly, flag it in the output but proceed with the rest.

### 2. Cluster

Group the extracted findings into candidate themes. Heuristics:

- Same named system, service, or dependency appearing in multiple contributing-factor lists (e.g. "the pricing service" in 4 of 12 postmortems).
- Same class of systemic cause across different incidents (e.g. "no ownership policy for cross-service calls," "no deprecation runbook," "no capacity-planning process for seasonal events").
- Same category of missing tooling (e.g. "no integration test for X," "no dashboard for Y").
- Same type of process gap (e.g. "no blocking review on schema changes," "no staging environment for Z").

A candidate theme needs at least 2 incidents to be worth including. 3+ is better. Singletons belong in "one-offs we noticed but can't call a trend."

### 3. Score

For each candidate theme, estimate:

- **Frequency** — N of M incidents. Report both.
- **Recency** — did the theme appear in the most recent incidents, or is it historical? Historical-only trends may already be resolved.
- **Severity weighting** — did any of the incidents in which this theme appeared reach SEV1 / high-impact? If so, flag it.
- **Action pattern** — did previous postmortems already propose actions addressing this theme? Did those actions ship? If the same systemic cause keeps appearing and corrective actions keep being written and not implemented, that's its own meta-finding.

### 4. Write the synthesis

Save to `./trends-<folder-name>.md`. Use this structure:

```markdown
# Trends synthesis — <folder-name>

**Source**: <N> postmortems in `<folder>`, spanning <earliest date> to <latest date>.
**Data-quality caveats**: <note any postmortems skipped or partially inferred — e.g. "3 of 14 postmortems lacked clear layer separation; findings from those were inferred and are marked [inferred] below.">

## Top recurring themes

### Theme 1: <short name, e.g. "Service ownership ambiguity">

**Frequency**: appeared in N of M incidents (list them by file name or short slug).
**Recency**: <most recent date / spans last quarter / historical only>.
**Severity weighting**: <any SEV1s? any prolonged outages? any external-facing impact?>

**What this looks like in practice**: <one paragraph, grounded in quotes or paraphrases from the postmortems. Name services, teams, or processes specifically.>

**Previous actions taken**: <did any postmortem propose a fix? Did it ship? If the same systemic cause keeps recurring despite proposed actions, say so — that's the meta-finding.>

**Recommended org-level intervention**: <one or two specific proposals. Not hedged. Not generic. Example: "Publish a service ownership map covering all production services and require that every service have a named owning team. Review quarterly. The last three incidents in which ownership ambiguity was a contributing factor would have been either prevented or detected earlier given this policy.">

### Theme 2: ...

<Same structure. Aim for 3–5 themes total. If you find fewer than 3, say so — don't invent themes to hit a target count.>

## One-offs worth noting

<Findings that appeared only once but seem noteworthy. Lower confidence, shorter treatment — a sentence or two each. These are leading indicators, not trends.>

## Data-quality observations about the postmortem process itself

<If the corpus reveals issues with how postmortems are being written — hedged findings, all-Prevent action lists, missing "what went well" sections, shallow systemic-cause layers — note that here. It's a trend about the process, not about the systems. Example: "8 of 14 postmortems in this corpus have action lists that are 100% Prevent-tagged, suggesting the team is under-investing in Detect / Mitigate / Respond capabilities.">

## Recommended next step

<Pick one:
- Hand this trends document to <team / forum / leadership review> for investment decisions.
- Commission a follow-up extended RCA on the top recurring theme, using this trends doc as the factual base.
- Use this to inform the next quarter's reliability roadmap.

Give a one-sentence reason.>
```

## Things to avoid in trends mode

- **Don't invent patterns.** If a candidate theme has only 2 weak connections, it goes in "one-offs." Overclaiming trends is how this mode turns into pattern-matching theater.
- **Don't re-analyze individual incidents.** If a postmortem itself is shallow, note it and move on. Don't rewrite it — that's not the job of trends mode.
- **Don't output a generic engineering-best-practices document.** The trends have to be grounded in specific evidence from the specific postmortems. Generic advice ("invest in observability") is useless; specific advice ("the pricing service appeared as a contributing factor in 5 incidents and has no circuit breaker despite two proposed actions in 2025-Q3 and 2025-Q4") is actionable.
- **Don't hedge the recommendations.** The whole point of doing the synthesis is to make the investment case. Hedged recommendations ("perhaps consider investing in...") defeat the purpose.
- **Don't ignore the meta-finding.** If the corpus shows the team is writing shallow postmortems, or failing to ship corrective actions, or over-tagging Prevent — that's a finding about the RCA process itself, and it belongs in the trends document.
