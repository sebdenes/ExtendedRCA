# Contributing to extended-rca

Thanks for wanting to help. This project exists because most RCAs stop too early, and the best way to make it better is to stress-test it against real incidents.

## Ways to contribute

**High-value, low-barrier:**
- Add a real-world (anonymized) incident scenario to `examples/`. A timeline and a brief narrative are enough to be useful. A shallow draft + the deepened version you'd hope `/rca deepen` produces is even better.
- Improve the prompting guidance in `skills/extended-rca/references/*.md`. If you've actually done many postmortems and think a trap is missing from `five-whys.md`, open a PR.
- Tighten or extend the artifact template in `skills/extended-rca/references/artifact-template.md`. Sections that consistently get written poorly are candidates for better scaffolding.

**Core changes:**
- Changes to the workflow phases, the artifact template, or the fishbone categories — open an issue first. These are opinionated choices; we'd rather discuss before you spend time.

## Style

- Markdown: one sentence per line is fine but not required. Keep headings terse.
- Prose in skill files: second-person ("you"), imperative mood, no filler.
- Resist softening systemic findings. Vague phrasing is the failure mode this skill is designed to resist — don't reintroduce it in the guidance.

## PRs

- Small PRs are better than big ones.
- Include a brief description of *why*, not just *what*.
- Link to the issue if there is one.
- If you're adding an example or case study, confirm it's been anonymized and name the license of any source material.

## Code of conduct

Be kind. This project is about learning from mistakes — the subject matter rewards intellectual humility.
