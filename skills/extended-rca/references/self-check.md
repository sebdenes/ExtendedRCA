# Self-check — Phase 6

Run this checklist against the draft artifact before handing it to the user. Every "fail" requires a fix in place. Self-check is not a ceremony — it's the last line of defense against the failure modes this skill exists to resist.

Go through the checks in order. A draft doesn't pass until every check is either green or has been explicitly, specifically acknowledged in the "open questions" section.

## Structural checks

**1. All four causal layers populated?**

The artifact has distinct sections for Trigger, Proximate cause, Contributing factors, and Systemic / latent causes. Each section has real content or the explicit string "None identified." An empty section is worse than "None identified" — it reads as though the layer was skipped.

Fail case: contributing and systemic layers are smooshed together, or only one of the four is filled in.

Fix: separate them. If you genuinely have no content for a layer, write "None identified" and move on — don't pad.

**2. Actions span at least three of Prevent / Detect / Mitigate / Respond?**

Count the action tags. If they all say "Prevent," the postmortem is quietly assuming this class of incident will never happen again. That's almost always wrong.

Fail case: four Prevent actions, one Detect, zero Mitigate, zero Respond.

Fix: generate at least one Mitigate action (how do we reduce impact next time?) and one Respond action (how do we speed up the human response?). If you genuinely can't, note it in "open questions."

**3. Every action traces to a specific finding?**

Pick a random action. Follow the thread backward: which contributing factor or systemic cause does it address? If you can't name the finding, the action is padding.

Fail case: "Action: improve observability." Traces to nothing specific.

Fix: either attach the action to a specific finding (e.g. "addresses Contributing factor #2: detection lag of 9 minutes") or delete it.

**4. Causal chain doesn't terminate at human error or a single commit?**

Read each why-chain end-to-end. If the terminal answer is "Alice merged the bad commit" or "the on-call didn't check the alert," the chain stopped too early.

Fail case: "Root cause: engineer pushed a bad config." End of chain.

Fix: ask *why* that was possible. What made it likely? Why wasn't it caught? What was the review state? What were the incentives? Keep going until you land on a structural condition — a missing policy, a tooling gap, an ownership ambiguity, an incentive that works against correctness.

## Language checks

**5. No hedged systemic findings?**

Scan systemic findings for hedge words: "could be improved," "perhaps," "might consider," "some communication issues," "we should probably." Hedging is how systemic findings get softened into uselessness.

Fail case: "Communication between teams could be better."

Fix: rewrite specifically. Name the teams, name the gap, name the evidence. "Payments and pricing teams have no shared channel for breaking changes, and the last three cross-team incidents all involved undiscovered breaking changes."

**6. No blame-ful phrasing toward a person or team?**

Search for patterns like "the engineer who...," "[NAME] should have...," "the on-call failed to...," "the team was careless." These are blame, not analysis.

Fail case: "Alice didn't test the migration carefully enough."

Fix: rewrite system-first. "The migration tooling doesn't require dry-run output before merge, so a human reviewer catching a missed test case is the only safety net — and that safety net is cognitively expensive under time pressure."

**7. Concrete nouns and verbs, not vague?**

Ban: "the system," "something went wrong," "the issue," "problems occurred." Require: service names, endpoint names, config keys, commit hashes, flag names, specific error types.

Fail case: "The checkout system experienced issues."

Fix: "The `checkout-worker` service began panicking at 14:04 UTC when `pricing-client.GetPrice` returned `nil` for 3% of requests. Panics crashed the worker, and the Kubernetes restart loop added 40s of recovery lag per instance."

## Content checks

**8. At least three hypotheses considered (even if two were ruled out)?**

The "Hypotheses considered and ruled out" section exists to resist premature convergence. If it's empty or only lists the winning hypothesis, Phase 2 was skipped.

Fail case: only the accepted cause appears.

Fix: list two or three plausible alternative narratives, what evidence you weighed, and why they were ruled out. Brief is fine — a sentence per hypothesis.

**9. "What went well" section is real, not filler?**

Check that this section names specific things that worked — detection, response, mitigation, communication, recovery — not generic praise ("the team responded professionally").

Fail case: "Everyone did a great job."

Fix: name the pattern. "Detection fired within 90s of first failed request because the synthetic monitor on `/checkout` had p99 alert coverage. That pattern should be preserved."

**10. "Open questions" honest about uncertainty?**

If any contributing or systemic finding is supported by only thin evidence, it should be in "open questions," not stated as established fact.

Fail case: stating "service ownership is the systemic cause" with no supporting evidence, and no "open questions" section.

Fix: demote weak findings to "open questions" with a one-line reason. ("Service ownership ambiguity *may* be a factor; we've observed one related incident in the last quarter but the pattern isn't yet clear enough to call it systemic.")

## How to apply the checklist

Don't just scan and move on. For each failed check, fix the artifact in place, then re-run the affected checks — fixes sometimes introduce new failures (e.g. rewriting a hedged finding may also require adding a Detect action to match the newly-specific gap).

When you're done, tell the user which checks passed and which required fixes. Transparency about the verification pass is part of the rigor.
