# Running the eval suite

The eval suite is designed to be run manually in Claude Code by a human or reviewer. There is no test runner — each case is an invocation and a structured comparison against expectations.

## When to run

- Before merging any PR that touches `skills/extended-rca/SKILL.md`, `commands/rca.md`, or any file in `skills/extended-rca/references/`.
- Before publishing a new release.
- After any change to the artifact template.
- Quarterly, as a baseline sanity check.

## Protocol

1. **Start a fresh Claude Code session** in this repo's root directory. Starting fresh avoids context carry-over affecting the output.

2. **For each case in `cases.md`**:
   - Copy the invocation verbatim into Claude Code.
   - Wait for the full output.
   - Compare the output against the expected properties, using the rubric in `rubric.md`.
   - Score each rubric check pass / partial / fail.
   - Record the score for the case and any notes on surprises.

3. **Re-run any case that fails the first time.** LLM outputs have variance. Count a case as a pass if it passes 2 of 3 runs.

4. **Write results to a file**: `evals/results-<YYYY-MM-DD>.md`. Include:
   - Per-case pass / partial / fail verdict.
   - Per-case rubric scores.
   - Any cases that required re-runs.
   - Any regressions vs. the previous results file.

## Baseline results to compare against

The first time the suite is run against a version of the skill, record that version's results as the baseline. Subsequent runs compare against the most recent green baseline.

A change that regresses any case by more than one rubric point (or flips any must-flag assertion) needs explicit justification in the PR description. Unexplained regressions are blockers.

## Example results-file layout

```markdown
# Eval run — 2026-04-19

## Baseline version
Skill commit: <sha>

## Per-case results

### Case 1 — Review shallow login
- Attempt 1: pass (grade 10/30, flagged all 4 must-flag issues)
- Verdict: pass

### Case 2 — Review deepened login
- Attempt 1: pass (grade 27/30)
- Verdict: pass

### Case 3 — Deepen capacity surge
- Attempt 1: partial (12/15 — P7 weak; systemic cause named but not specific enough)
- Attempt 2: pass (14/15)
- Verdict: pass (2 of 2 so far)

### Case 4 — Deepen cross-team
- Attempt 1: pass (13/15)
- Verdict: pass

### Case 5 — Hypotheses only
- Attempt 1: pass (H1–H4 all pass, 4 hypotheses across 4 fishbone categories)
- Verdict: pass

### Case 6 — Full RCA fresh narrative
- Attempt 1: pass (13/15)
- Verdict: pass

## Regressions vs. previous baseline
None.

## Notes
- Variance on Case 3 is higher than other cases; consider tightening the "systemic cause" guidance in SKILL.md if this persists.
```

## Budget

A full run of the 6 cases takes roughly 20–30 minutes of hands-on time if cases pass on the first try, 45–60 minutes if re-runs are needed. This is a deliberate trade-off: manual + fast > automated + brittle, given the small suite size.
