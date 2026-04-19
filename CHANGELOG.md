# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] — 2026-04-19

### Added
- Initial release as a Claude Code plugin.
- `/rca` slash command with five modes: default, `deepen`, `review`, `hypotheses`, `trends`.
- `extended-rca` skill with:
  - `SKILL.md` — philosophy, the 6 universal phases (Phases 1–5 + Phase 6 self-check), tone, and anti-patterns.
  - `references/five-whys.md` — 5-whys without the usual traps.
  - `references/fishbone.md` — 6 software-specific categories (People / Process / Code / Infrastructure / Data / External).
  - `references/artifact-template.md` — the postmortem Markdown template.
  - `references/self-check.md` — Phase 6 verification checklist applied before handover.
  - `references/review-rubric.md` — 10-axis / 30-point rubric for `/rca review`.
  - `references/trends-synthesis.md` — cross-incident synthesis workflow for `/rca trends`.
- Five worked examples in `examples/`:
  - `login-outage/` (Code / Process)
  - `capacity-surge/` (Infrastructure / Process)
  - `silent-data-corruption/` (Data / People)
  - `third-party-timeout/` (External / Code)
  - `cross-team-breaking-change/` (Process / People)
- Eval harness in `evals/` — six cases covering review, deepen, hypotheses, and full-RCA modes, plus rubrics for each output type and a manual execution protocol.
