# 5-Whys, Done Properly

The 5-whys technique is borrowed from Toyota's production system and has become the default causal-analysis tool in software RCA. It's genuinely useful, but it has two reliably-recurring failure modes. Both are about *where you stop*.

This reference explains what good and bad 5-whys chains look like, so you can produce the first kind in the causal-chain phase of an RCA.

## The basic shape

Start with the observable symptom. Ask "why did that happen?" Write down the answer. Ask why *that* happened. Repeat. The "5" is a convention — you stop when you hit something structural, not when you've counted to 5.

## Failure mode 1 — Stopping at a human

The most common mistake is ending the chain at a single person's action:

> Why did the deploy fail? *Alice merged bad code.*
> Why did Alice merge bad code? *She didn't run the tests locally.*
> Why didn't she run the tests locally? *She was in a hurry.*

This chain is three links deep but it has no systemic insight. "She was in a hurry" is a symptom of something, not a root cause. The chain should continue:

> Why was she in a hurry? *The deploy had to go out before the marketing campaign launch.*
> Why did the deploy have to go out right then, without time for local tests? *Our release train doesn't have a slot for urgent deploys with compressed testing; every change goes through the same pipeline regardless of urgency.*
> Why doesn't our release train have that slot? *We haven't defined tiers of deploy urgency, so all changes are batched together and ad-hoc urgent deploys get squeezed into the regular process without adjustment.*

Now you've landed somewhere actionable at the system level: "define deploy-urgency tiers with appropriate process adjustments." That's a fix to the *system*. "Tell Alice to run tests" is a fix to an individual, and the next engineer in the same situation will do the same thing.

**Rule of thumb: if the terminal "why" still names a person, you haven't gone deep enough.**

## Failure mode 2 — Single-threaded reasoning

Real incidents are rarely single-cause. The clean-line 5-whys chain often hides the messier structure. Watch for places where two or more contributing chains converge on the same impact:

> **Chain A:** The deploy introduced a nil-handling bug → the code reviewer didn't catch it → the review rules don't route pricing-client changes to pricing-service reviewers → we don't have ownership policies for cross-service calls
>
> **Chain B:** The alert fired 9 minutes late → the alerting threshold is calibrated to 5% error rate → this incident only hit 3% → our alerting philosophy assumes volume, but this service has low baseline traffic and 3% is a meaningful spike for it
>
> **Chain C:** Rollback required a manual approval → the approval rule was added two years ago after an unrelated incident → nobody re-evaluated whether it still made sense

Each chain has its own structural cause. A 5-whys writeup that hides Chains B and C behind "the deploy introduced a bug" loses 2/3 of the actionable learning.

When you find yourself writing a single 5-whys chain, stop and ask: "Were there other contributing conditions that, if changed, would have prevented or reduced the impact?" Usually yes.

## Failure mode 3 — False depth

The opposite problem: a chain that keeps asking why but stays shallow. The "whys" become increasingly abstract without landing anywhere you can act on.

> Why did the deploy fail? *A bug.*
> Why was there a bug? *Software is complex.*
> Why is software complex? *It just is.*

This is a yellow flag for two things: either the initial symptom wasn't precise enough (say "the deploy crashed the checkout worker" not "the deploy failed"), or the chain has lost its anchor to the specific incident and drifted into generic software philosophy.

If a chain starts to feel abstract, re-anchor: "what specifically about *this* change, in *this* service, at *this* time, made this happen?"

## Endpoints that suggest you've gone deep enough

Good terminal whys tend to have one of these shapes:

- **Ownership / policy gap** — "there's no owner for X, so Y wasn't maintained"
- **Design assumption that no longer holds** — "this service was designed for a world where upstream latency was bounded, and now it isn't"
- **Incentive misalignment** — "investing in Z would reduce incident rate, but it loses every prioritization bet to feature work"
- **Knowledge gap** — "the team that made this change doesn't have the historical context that would have made the risk visible"
- **Missing safeguard** — "the safeguard that should have caught this doesn't exist / was silently disabled / isn't tested"
- **Process gap** — "there is no review step / testing step / rollout step that would have caught this class of issue"

These are endpoints that point at something you can actually change — a policy, a process, an investment, an ownership assignment. Those are the fixes that prevent the next incident of this class.

## Combining 5-whys with fishbone

A useful discipline: before you start the 5-whys chain, look at your fishbone categorization (see `fishbone.md`) and check whether your chain covers multiple categories. A chain that only threads through "Code" is missing People, Process, Infrastructure, Data, and External chains. You don't need to build a 5-whys chain for *every* category, but you should notice if one category has multiple contributing factors and build the chain for the important ones.

## Writing the chain in the artifact

In the final RCA document, present why-chains as a short labeled sequence under the "Systemic causes" section. Don't write paragraphs — write the chain. Reader should be able to scan the "because" links and see the logic.

**Example of how it reads in the artifact:**

> **Why the 14:02 deploy panicked checkout workers:**
>
> The pricing-service client didn't handle nil responses
> → because the code path was added under time pressure with no nil-response test
> → because the pricing-service client's test suite has no fuzzing for upstream-response shape
> → because there's no policy about nil-response handling for external-service calls, and no owner for the pricing-service client after the refactor last year
> → because service ownership boundaries between payments and pricing teams became fuzzy after the reorg, and no one has revisited ownership assignments

That chain is 5 links deep, lands at an ownership/policy cause, and suggests a concrete corrective action (revisit service ownership, define a nil-response policy). That's what a 5-whys chain should look like.
