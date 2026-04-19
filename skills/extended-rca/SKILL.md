---
name: extended-rca
description: Run a thorough, methodology-aware post-incident root cause analysis on a software outage or production incident. Use this skill whenever the user wants to understand *why* something really broke — after the incident is resolved and they hand over some mix of incident summary, timeline, logs, or a draft postmortem. Also use it when they say things like "do an RCA," "5 whys on this," "extended postmortem," "fishbone this incident," "root cause analysis," "contributing factors," or "what really went wrong." The skill combines narrative 5-whys and fishbone categorization with explicit layer separation (trigger / proximate / contributing / systemic) and corrective actions tagged by type (Prevent / Detect / Mitigate / Respond), and produces a structured engineering-grade writeup. Distinct from active incident triage — use `engineering:incident` / `engineering:incident-response` for that.
---

# Extended Root Cause Analysis

## Why this skill exists

Most RCAs stop too early. The team finds a plausible proximate cause ("the deploy introduced a bug"), writes it up, schedules one ticket, and moves on. Six weeks later the same class of incident recurs.

A good extended RCA goes deeper in three ways at once:

1. **Vertical depth** — it keeps asking *why* past the first plausible answer, until it lands on a systemic cause (design, process, or organizational) that actually explains why the proximate cause was possible.
2. **Horizontal breadth** — it considers several competing hypotheses and evaluates them against evidence, rather than committing to the first story that fits.
3. **Layer separation** — it distinguishes trigger from proximate cause from contributing factors from systemic causes, and writes corrective actions at each layer, not just the surface.

Under time pressure, engineers reach for the first plausible story. This skill exists to resist that pull and produce analyses a team can actually learn from.

## When to use this skill vs. adjacent skills

Use this skill when the incident is **already resolved** and the user wants depth. For active triage or in-the-moment incident response, point them at `engineering:incident` / `engineering:incident-response` instead. If they just want a short status-page blurb, this is overkill — offer a one-paragraph summary instead.

## How the skill works

Look at what the user actually has and what they want from it:

- **A fresh incident (summary, timeline, logs, or any combination)** — apply the universal phases below (5-whys + fishbone categorization + layer separation + tagged actions + self-check).
- **A draft postmortem that stops too shallow** — the goal is usually to deepen it. Read what they have, identify which layers are missing or confused (most often the contributing-factors and systemic-causes layers), and extend it. Make the diff visible — don't silently rewrite their work.
- **A postmortem they want graded, not rewritten** — use the review rubric in `references/review-rubric.md`. Output is a findings list, not a rewritten artifact. Distinct from `deepen`: review critiques, deepen extends.
- **A folder of past postmortems they want synthesized** — use `references/trends-synthesis.md` to cluster recurring systemic themes across incidents. The output is a trends document, not a per-incident RCA.
- **Just competing hypotheses, without a full RCA** — stop after Phase 2. The output is the fishbone-categorized hypothesis list with confirm/disconfirm evidence notes.

The slash command's modes (`deepen`, `review`, `trends`, `hypotheses`) map to these shapes. Default mode runs the full universal phases.

## The universal phases

The analysis progresses through these phases. They build on each other.

### Phase 1 — Establish the facts

Before reasoning about causes, pin down what actually happened. Write a clean version of:

- **Impact**: who was affected, how, for how long, in measurable terms where possible (e.g. "17% of checkout requests failed between 14:02 and 14:47 UTC, about $X in dropped orders").
- **Timeline**: a tight chronology of detection, response, and resolution events — not a story, a sequence of timestamped events.
- **Blast radius**: which services, regions, user segments, and downstream dependencies were affected.

If you can't construct any of these with confidence, stop and ask one or two targeted questions. A shaky factual base will poison everything downstream. But don't over-interview — work with what the user has and flag real gaps in the final artifact.

### Phase 2 — Generate multiple hypotheses (use fishbone if it helps)

This is the single most important habit of a good RCA: **do not commit to one story yet.** List at least three candidate causal narratives that could plausibly explain the symptoms — even when one feels obviously right. Confirmation bias is real and extremely hard to notice from the inside.

A helpful trick when the incident touches multiple concerns: generate hypotheses by category, fishbone-style. The classic "6 M's" from manufacturing don't quite fit software, so use these categories instead. Full details and prompting questions for each category are in `references/fishbone.md` — read it when you need to break out of tunnel vision.

- **People** — who was on call, what did they know, what context was missing
- **Process** — how changes get reviewed, deployed, tested, rolled back
- **Code** — what the code actually does vs. what was assumed
- **Infrastructure** — compute, storage, network, shared resources, quotas
- **Data** — schema, volume, distribution, new patterns, poisoned inputs
- **External dependencies** — upstream services, third-party APIs, providers

Not every category will have content for every incident. But forcing yourself to ask "what could People have contributed here? Process? Data?" uncovers hypotheses you'd otherwise miss.

For each hypothesis, note briefly what evidence would **confirm** it and what would **disconfirm** it. Then weigh the actual evidence. Keep the losing hypotheses in the final artifact as a short "considered and ruled out" section with the disconfirming evidence — this prevents re-exploration later and shows the analysis was rigorous.

### Phase 3 — Build the causal chain (5-whys, done properly)

For the leading hypothesis (or hypotheses — sometimes there are two converging chains), build a why-chain. The classic 5-whys is a useful shape, but it has two failure modes this skill actively resists. They're covered in depth in `references/five-whys.md`; the short version:

- **Stopping too early.** If the final "why" still points at a single commit, a single human action, or a single process step, keep going. Systemic causes live deeper.
- **Single-threaded reasoning.** Real incidents usually have multiple contributing chains that converge. If the narrative reads as one clean line, you're probably hiding the messier structure.

A chain has reached enough depth when the terminal "why" points at something structural — a design choice, an ownership gap, a policy that doesn't exist, an incentive that works against correctness. "Alice merged a bug" is a trigger, not a root cause. "No test existed that could exercise real traffic patterns against staging, because investing in that tooling has lost every prioritization bet against feature work for the last three quarters" is a structural cause.

### Phase 4 — Separate layers of causation

Structure findings into four distinct layers. Confusing them is the most common RCA failure mode — every layer gets smooshed into "the root cause" and the real systemic learning is lost.

- **Trigger** — the specific event that made the incident visible (e.g. "the 14:02 deploy").
- **Proximate cause** — the mechanism by which the trigger produced the impact (e.g. "the new code path didn't handle nil responses from the pricing service, which caused panics that crashed the checkout worker").
- **Contributing factors** — conditions that made the proximate cause more likely or more severe, but weren't themselves sufficient (e.g. "incomplete test coverage on the pricing-service client; alerting lag of 9 minutes; rollback required a manual approval step").
- **Systemic / latent causes** — underlying organizational, process, or design conditions that allowed the contributing factors to exist (e.g. "service ownership boundaries between the payments and pricing teams are fuzzy; there's no policy for how external-service calls should handle nil responses").

Every significant incident has content at every layer. If the user's draft stops at trigger + proximate cause, your contribution as an extended RCA is most visible in the contributing and systemic layers.

### Phase 5 — Generate corrective actions, tagged by type

Actions are more useful when tagged by what they do. Use this taxonomy:

- **Prevent** — stop this class of incident recurring (e.g. "add contract tests for nil responses from the pricing service")
- **Detect** — catch it faster next time (e.g. "add an SLO alert on pricing-service error rate")
- **Mitigate** — reduce impact when it does happen (e.g. "make the pricing-service client fail open to a cached default")
- **Respond** — speed up the human response (e.g. "automate the rollback step that currently requires manual approval")

A healthy set of actions has content in at least three of these buckets. A list of only "Prevent" actions is a red flag — it implicitly assumes the team will never fail again, which is not a safe assumption for any serious production system. For each action, note the layer of causation it addresses, an owner (or "TBD"), and whether it's must-do or should-consider. Three strong actions beat ten weak ones.

### Phase 6 — Self-check before handing over

Before you declare the artifact done, run a verification pass against `references/self-check.md`. The checklist catches the failure modes this skill is designed to resist:

- Did a causal chain terminate at "human error" or a single named person's action? Keep going.
- Is any systemic finding hedged ("communication could be improved," "we should consider")? Rewrite it to be specific, non-blameful, and actionable.
- Do the corrective actions span at least three of Prevent / Detect / Mitigate / Respond?
- Does every corrective action trace to a specific finding?
- Are the four causal layers (trigger / proximate / contributing / systemic) all populated, or explicitly marked "None identified"?
- Does the writing name specific services, endpoints, commits, configs — not vague verbs like "the system broke"?
- Is any phrasing blame-ful toward a person or a single team?

If any check fails, fix it before handing the artifact over. The self-check is not optional; it's what distinguishes an extended RCA from a shallow one that happens to follow the template.

## The written artifact

Produce a Markdown document using the template in `references/artifact-template.md`. The structure there is deliberate — each section exists to resist a specific failure mode:

- Trigger / proximate / contributing / systemic layers → resists layer-confusion
- "Hypotheses considered and ruled out" → resists premature convergence
- "What went well" → resists the negativity bias that makes every postmortem read as purely damning
- "Open questions / unknowns" → legitimizes admitting what the evidence doesn't support

Read the template before writing so you internalize the structure. Then fill it in using the findings from the phases above.

## Tone and framing

Write in the past tense. Be specific — name services, endpoints, commits, config keys, flag names. Avoid vague verbs ("the system broke," "something went wrong") — say what broke and how.

**Stay non-blameful in phrasing but not non-specific in content.** "The engineer who merged the bad commit" is bad style. "The change was merged without a reviewer familiar with the pricing client's nil-handling, because review-routing rules don't account for cross-service calls" is good style — it's specific, it names a process gap, and it points at something fixable.

When a contributing or systemic cause reflects poorly on a team or a decision, say it clearly. Softening systemic findings into vagueness ("communication could be improved") defeats the point.

## Anti-patterns to actively avoid

A good RCA is partly defined by what it refuses to do. Watch for these in your output and correct them:

- **Single-cause narrative.** Real incidents have multiple contributing factors. A clean one-line cause-and-effect is usually a sign of over-simplification.
- **"Human error" as a terminal cause.** Human error is a symptom of system design, not a root cause. If a chain ends at "person made a mistake," keep going: what made the mistake likely, undetectable, or catastrophic?
- **Actions without causes.** Every corrective action should trace to a specific finding. If you can't point to the finding, either the finding is missing or the action is unnecessary padding.
- **Hedged systemic findings.** "Communication could perhaps be improved" is not a finding. "Payments and pricing teams have no shared channel for breaking changes, and the last three cross-team incidents all involved undiscovered breaking changes" is a finding.
- **Omitting what went well.** Every postmortem should identify preserved patterns, not just things to fix.

## A note on uncertainty

Extended RCA is reasoning over incomplete information. Be honest about confidence. If the evidence only weakly supports a systemic cause, say so and put it in "open questions" rather than stating it as established. Overstating confidence in systemic findings is how RCAs turn into political instruments rather than learning instruments.

## Reference files

- `references/five-whys.md` — how to do 5-whys without falling into the usual traps
- `references/fishbone.md` — the 6-category framework adapted for software incidents
- `references/artifact-template.md` — the Markdown template for the final writeup
- `references/self-check.md` — Phase 6 verification checklist, applied before handing any artifact over
- `references/review-rubric.md` — grading rubric and output format for `/rca review`
- `references/trends-synthesis.md` — how to do cross-incident synthesis for `/rca trends`
