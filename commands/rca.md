---
description: Run an extended, methodology-aware root cause analysis on a resolved software incident — or review, deepen, hypothesize, or synthesize trends across postmortems.
argument-hint: "[mode] <incident description | path to postmortem | path to folder>"
---

# /rca

Run a thorough post-incident root cause analysis. Unlike a shallow RCA that stops at the first plausible cause, this command uses narrative 5-whys and fishbone categorization to produce a structured engineering-grade postmortem with clear layer separation (trigger / proximate / contributing / systemic) and tagged corrective actions.

This command always uses the **extended-rca** skill; do not try to answer without reading the skill first. The skill defines the workflow, the methods, and the artifact template. Read `skills/extended-rca/SKILL.md` and the relevant reference files before starting the analysis.

## Usage

```
/rca $ARGUMENTS
```

`$ARGUMENTS` is free-form. The user can paste an incident summary, a timeline, a draft postmortem, a path, or any combination. Expect underspecified inputs and ask one or two focused questions only when something critical is missing (impact, timeline, or scope).

## Modes

- **Default** — read `$ARGUMENTS` as an incident description and perform the full extended RCA. If inputs are incomplete, ask targeted questions before proceeding.
- `/rca deepen <path-to-draft>` — read an existing postmortem draft and extend it, adding whichever layers are shallow or missing (most often: alternative hypotheses, contributing factors, systemic causes, tagged corrective actions). Preserve the user's existing content; make the diff visible.
- `/rca hypotheses <incident summary>` — stop after Phase 2 (multi-hypothesis generation using fishbone categories). Useful when the user wants help thinking broadly before committing to a narrative.
- `/rca review <path-to-postmortem>` — grade an existing postmortem against the quality rubric **without rewriting it**. Returns a findings list: layer-by-layer assessment, specific issues (hedged findings, single-bucket action lists, terminal "human error"), and a recommended next step. Use `references/review-rubric.md`.
- `/rca trends <path-to-folder>` — ingest multiple postmortems from a folder and synthesize recurring systemic themes. Returns a trends document listing the top 3-5 recurring contributing or systemic causes, with evidence counts and recommended org-level interventions. Use `references/trends-synthesis.md`.

If no mode is specified, run the default flow.

## Workflow

Follow the phases described in the **extended-rca** skill's SKILL.md:

1. **Establish the facts** — impact, timeline, blast radius. Ask one or two questions if a critical piece is missing.
2. **Generate multiple hypotheses** — at least three, using the fishbone categorization.
3. **Build the causal chain** — 5-whys, done properly (see `references/five-whys.md`). Watch for stopping-too-early and single-threaded reasoning.
4. **Separate layers of causation** — trigger, proximate cause, contributing factors, systemic / latent causes. Every significant incident has content at every layer.
5. **Generate corrective actions, tagged by type** — Prevent / Detect / Mitigate / Respond. A healthy set has content in at least three buckets.
6. **Self-check the artifact** — run the verification pass (see `references/self-check.md`) before handing over. Catch hedged findings, blame-ful phrasing, single-bucket action lists, and terminal "human error" causes. Fix them in place.
7. **Write the artifact** — use the template in `references/artifact-template.md`. Save the output as a Markdown file and tell the user the path.

In `review` and `trends` modes, replace Phases 3-7 with the mode-specific workflow in the corresponding reference file.

## Output

A Markdown document following the appropriate template:
- Default / deepen / hypotheses → postmortem using `references/artifact-template.md`, saved to `./postmortem-<incident-name>.md` by default.
- Review → findings list, saved to `./review-<postmortem-name>.md`.
- Trends → synthesis document, saved to `./trends-<folder-name>.md`.

Always confirm the save location with the user if you're not sure.

## Things to avoid (see the skill for details)

- Single-cause narratives. Real incidents have multiple contributing factors.
- Terminating a why-chain at "human error" or at a single person's action.
- Hedged systemic findings like "communication could be improved."
- Action lists that contain only "Prevent" items.
- Silently rewriting the user's draft in `deepen` mode. Make the diff visible.
- Grading a postmortem in `review` mode by rewriting it. Review returns findings; deepen rewrites.
