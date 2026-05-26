# Elicitation Playbook

How to run the discussion so it earns its keep. The goal is to extract the requirements the user
*hasn't articulated* — the boundaries, failure modes, and decisions hiding inside "build X" —
without turning the session into a questionnaire.

## How to use `AskUserQuestion` well

`AskUserQuestion` is for **decisions**, not for harvesting facts you could read or infer.

- **Every question is a fork.** Ask only where the answer changes what gets built. If you already
  know the likely answer, propose it as the recommended option and let the user veto — don't ask
  open-endedly.
- **Give real options with trade-offs.** A good option names a concrete choice *and* its
  consequence ("Recompute when a rule changes — always current, but past results shift"). The user is
  choosing between futures, not filling in a blank.
- **Lead with your recommendation.** Put the option you'd pick first and mark it
  "(Recommended)", with the reason. This turns a quiz into a review.
- **Batch related decisions** (2–4 per round) so the user sees a coherent slice, then move on.
- **Carry the consequence forward.** When they pick, reflect what that decision now implies for
  the rest of the design before asking the next thing.

### Banned (chore) questions vs sharp questions

| Chore (don't) | Sharp (do) |
|---|---|
| "What should the system do?" | "When two requests for the same key arrive at once, does the second wait, fail, or overwrite?" |
| "Is performance important?" | "What's the p99 latency budget, and what's the right behaviour when you'd blow it — queue, shed, or degrade?" |
| "Any other requirements?" | "A back-dated request arrives for a date before the current rule existed — what answer is correct?" |
| "Do you want validation?" | "Which inputs are trusted (came from an authenticated internal caller) vs must be validated at this boundary?" |
| "Should it be scalable?" | "What's the expected volume in 12 months, and which part breaks first when it 10×s?" |

The pattern: replace a yes/no or open prompt with a **specific scenario at a boundary** that forces
a real decision.

## Read before you ask

Spend the first pass investigating, not interviewing. Pull from:

- The code and types (what already exists and constrains the answer).
- The ticket / request (what's already stated).
- Existing requirement docs and ADRs (what's already decided — don't relitigate).

Then ask only what remains genuinely open. Showing the user you've already read their code earns
the right to ask the hard questions.

## The question bank

Probe each category that applies. These are seeds — adapt the specifics to the domain. The
strongest questions name a concrete value or scenario, not an abstraction.

### Temporal & change-over-time (ask this for ANY rule/rate/policy system)
- When the rule/rate changes, do past computations stay stable or recompute? Who relies on
  stability?
- A request is back-dated to before the current rule existed — which version applies?
- A future-dated request arrives for a rule that changes next month — which version applies?
- How is a rule change rolled out: instant cutover, effective-dated, or shadow-run first?
- Can two valid answers exist for the same input at different points in time? Is that a bug or a
  feature?

### Boundaries & limits
- What are the min, max, zero, empty, and null cases for each input — and is each valid?
- What happens at the exact boundary (>= vs >)? Off-by-one here is a real bug.
- Largest realistic input? What overflows, truncates, or times out first?
- Precision: how is money rounded? How are dates/timezones/DST/leap years handled?
- First-ever and last-possible cases (first record, final instalment, end of a series).

### Failure & partial failure
- What happens when a downstream dependency is slow, down, or returns garbage?
- If the operation fails halfway, what state is left behind — and is that acceptable?
- Is the operation retryable? Is it idempotent under retry? What's the dedup key?
- What's the correct behaviour when you *can't* compute the right answer — fail loud, default, or
  degrade? Who must be told?

### Concurrency & ordering
- Can two operations on the same entity run at once? What's the desired outcome — serialise,
  reject, last-write-wins, merge?
- Do messages/events need ordering guarantees? What if they arrive out of order or twice?
- Is there a race between read and write (check-then-act)? How is it closed?

### Source of truth & consistency
- Which system *owns* each piece of data? Where's the authoritative copy?
- Is stale data acceptable here? For how long? Strong or eventual consistency?
- If two systems disagree, who wins and how is the disagreement detected?

### Trust & security boundaries
- Where is the trust boundary — which inputs are trusted vs must be validated here?
- Who is allowed to call this, and how is that enforced?
- What's the worst thing a malicious or buggy caller could make this do?
- What must never appear in logs/responses (secrets, PII)?

### Data lifecycle
- How long is each piece of data kept, and what triggers deletion?
- Is any of it PII / regulated? What handling does that class demand?
- Can you reconstruct *why* a stored value is what it is (provenance/audit)?

### Integration & contracts
- What's the exact shape of each API/event in and out, including the error contract?
- How does the contract evolve without breaking existing consumers (versioning)?
- What's the timeout/retry/circuit-breaker behaviour toward each dependency?

### Scale & operability
- Expected volume now and projected; which component is the first bottleneck?
- How will an operator know it's healthy? What's the alarm condition?
- What's the manual recovery path when it goes wrong at 3am?

### Scope edges (drive these to explicit non-goals)
- What might a reader assume is included that isn't?
- What's delegated to another service/team?
- What's "later, not now" vs "never"?

## Architecture-decision triggers

When the discussion hits one of these, you're no longer gathering a requirement — you're making a
decision that should become an ADR (see [documentation.md](documentation.md)):

- **Sync vs async** for an interaction.
- **Where authority lives** — which service owns a piece of state.
- **Consistency model** — strong vs eventual, and the trade-off accepted.
- **Build vs reuse** — new code vs an existing library/service.
- **Schema / contract versioning** strategy.
- **Caching / materialisation** of derived data and its invalidation.

Flag these explicitly to the user ("this is an architectural decision — let's record why"), get the
alternatives and the chosen trade-off on the record, and spin out an ADR.

## Knowing when to stop

Stop probing an area when: the happy path, the boundaries, the failure behaviour, and the
ownership/consistency questions all have answers — or have been deliberately logged as open
questions with an owner. Don't manufacture questions to seem thorough; an honest "this area is
fully specified" is the goal.
