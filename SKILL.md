---
name: audit-system-tradeoffs
description: Systems-reasoning skill for change review, architecture/design review, production debugging, incident analysis, and performance/reliability reasoning — with domain playbooks for Kubernetes, AWS, CI/CD, and databases. Use when evaluating a PR, design, or migration for preserved invariants, hidden trade-offs, event and input provenance, concurrency, retries, partial failure, ownership boundaries, architectural smells, or known distributed-system patterns (Saga, Outbox, CQRS, idempotency, circuit breakers); or when diagnosing an incident, a performance regression, or a reliability gap. Produces severity- and confidence-scored findings plus a minimal safe change.
license: MIT
metadata:
  author: David Uka
  version: "2.0"
---

# Audit System Trade-offs

Review the system before judging the proposed mechanism. Establish what must remain true, trace real events across boundaries, recognize the architecture you're actually looking at, and make every behavioral trade-off explicit.

This is a systems-reasoning methodology, not a linter or a single checklist.
Trade-off auditing (the five passes below) is the default mode and the
backbone every other mode reuses — but pick the mode that matches what
you're actually looking at before running the passes.

## Choose Your Mode

| You're looking at | Mode | Use |
|---|---|---|
| A PR, patch, or proposed mechanism | Change Review | Pass 1-5 below |
| A design doc, RFC, or architecture with no diff yet | Design Review | Pass 1-5 below; treat "the proposal" as the mechanism from Pass 1 onward |
| A schema/data migration, or a monolith-to-service / provider-swap migration | Migration Review | Pass 1-5 below; Pass 1 invariants and the Pass 4 ledger are the anchors, plus `references/domain-databases.md` for schema migrations |
| A live or reported problem, an outage, a bug you can't explain yet | Incident / Debugging | `references/incident-debugging.md` |
| "Is this fast/reliable enough," a regression, a scaling or SLO question | Performance / Reliability | `references/performance-reliability.md` |
| Kubernetes, AWS, CI/CD, or a database is involved, in any mode above | Domain playbook | Load the matching `references/domain-*.md` alongside whichever mode you're in |

All modes share the same Evidence Hierarchy, System Smells, and Assumptions
to Never Make defined below — those aren't specific to code review, they're
how you decide what's actually true about a system.

## Core Rule

Do not accept a local fix until it proves that it preserves higher-priority system behavior.

Use this reasoning order:
current behavior and invariants
  -> problem being solved
  -> proposed mechanism
  -> affected callers and boundaries
  -> concurrency and failure behavior
  -> trade-offs and alternatives
  -> recommendation

## Triage First: What Not to Review Yet

Do not spend review budget on formatting, naming, style, lint, or comment
wording until Pass 1-4 below are done. A perfectly formatted change that
violates an invariant is still a blocker; a readability nit in a change that
preserves every guarantee is not. If time runs out, invariants and ownership
must be covered — style is the part it's fine to skip.

## Pass 1: Establish Invariants

Read the existing implementation and identify behaviors that users or dependent systems rely on.

Ask:

- What works today and must continue working?
- Which events, updates, or records must never be lost?
- Which ordering, immutability, security, and human-review guarantees exist?
- Which behavior is intentionally manual or automatic?
- Which shared paths have callers outside this feature?

Write the invariants as observable statements, not implementation details. For example: Every accepted service update eventually reaches the platform chart is an invariant; Use a concurrency group is a mechanism.

Do not infer an invariant solely from comments. Confirm it against the evidence hierarchy below.

## Pass 2: Trace Provenance and Ownership

Trace one normal event end to end:
trigger -> payload producer -> transport -> consumer -> state mutation -> completion signal

For every important value, answer:

- Who creates it?
- Is it human input, machine input, derived state, or trusted metadata?
- What is its source of truth?
- Who validates it?
- Which component is allowed to mutate it?

Check that responsibilities live with the component that has the required state and authority. Do not move controller behavior into CI, cluster-owned secrets into GitHub Actions, or environment state into registry inference without an explicit reason.

If the change touches Kubernetes, AWS, CI/CD, or a database, read the
matching file in `references/domain-*.md` now — each covers the failure
modes generic review misses (e.g. reconciliation loops must be
level-triggered, DynamoDB GSIs are always eventually consistent, migrations
must survive a rolling deploy).

## Pass 3: Exercise Time and Failure

Evaluate more than the happy path. Trace:

1. Two valid events arriving together.
2. A retry after partial success.
3. A delayed or duplicated event.
4. A failure between an external side effect and state persistence.
5. An unsupported or malformed machine input.

Verify actual platform semantics instead of trusting names such as cancel-in-progress: false, idempotent, atomic, or retry.

For concurrency, identify:

- The collision domain created by the key.
- How many runs may be running and pending.
- Whether pending work is queued, replaced, or canceled.
- Whether payloads become stale while waiting.
- Which workload bears the delay or failure cost.

For retries, distinguish safe replay from merely repeating commands. Confirm the exact identity used for deduplication and the terminal acknowledgement state.

If the mechanism resembles a known distributed-system pattern (Saga,
Outbox, Inbox, CQRS, Event Sourcing, Leader Election, Reconciliation, State
Machine, Work Queue, Idempotency Key, Circuit Breaker, Bulkhead,
Backpressure), name it and check it against the guarantees that pattern is
supposed to provide — see `references/distributed-patterns.md`. A change
that half-implements Outbox or Saga is a specific, nameable bug, not a
vague "could lose data" note.

## Pass 4: Build a Trade-off Ledger

For each meaningful change, record:

| Question | Required answer |
|---|---|
| Problem | What concrete failure or requirement does this solve? |
| Gain | What becomes safer, faster, or possible? |
| Cost | What existing behavior becomes slower, weaker, or more complex? |
| Cost owner | Which team, workflow, or user experiences that cost? |
| Alternative | What smaller solution was considered? |
| Decision | Keep, modify, revert, or escalate? |

Treat an uncommunicated trade-off as an unresolved design decision. Prioritize primary system behavior over optional or diagnostic workflows unless requirements explicitly say otherwise.

Always ask:

> What existing good behavior becomes weaker because of this change?

## Pass 5: Justify Every Component

For each new file, script, controller, CRD, workflow job, secret path, and validation block, state in one line:
what it does + why the flow needs it

Remove a component when its behavior is already owned reliably elsewhere. Keep defensive validation when removing it would allow a false success, cross-boundary trust failure, incorrect replay, or destructive ambiguity.

Do not reduce lines of code by deleting safety semantics. Prefer reducing duplicated orchestration, unused outputs, redundant inputs, compatibility branches without real callers, and logic placed in the wrong owner.

## System Smells

Scan the change for these independent of whatever specific bug you find —
they're patterns senior engineers recognize on sight and often predict a bug
before you've traced the exact mechanism:

- **Orchestration duplicated** — the same coordination logic (retry, ordering, dedup) reimplemented in two places instead of one owner.
- **Hidden source of truth** — a cache, derived field, or copy is read as if it were authoritative.
- **Circular ownership** — component A's output is validated by B, whose output is validated by A.
- **Temporal coupling** — correctness depends on step X finishing before step Y with no explicit wait/signal enforcing it.
- **Implicit dependency** — behavior relies on another component's internals, timing, or undocumented default.
- **Retry amplification** — a retry at one layer multiplies into N retries at a layer below (e.g. client retry x per-item retry).
- **Fan-out explosion** — one event triggers an unbounded or unthrottled number of downstream calls.
- **State synchronization** — two stores must agree and nothing keeps them consistent except "it usually works."
- **Distributed transaction** — an operation needs atomicity across two systems and gets it from hope, not a pattern (Saga/Outbox) or a real two-phase mechanism.
- **Configuration drift** — behavior differs across environments due to config nobody is tracking as a diffable artifact.

## Assumptions to Never Make

These are the specific reasoning mistakes that produce false confidence.
Do not do these:

- Do not accept a fix because its tests pass — confirm the tests exercise the actual invariant, not just the happy path.
- Do not assume a retry is safe — confirm the operation is idempotent under the actual retry trigger.
- Do not assume "atomic" without confirming which operations are inside the same transaction/lock.
- Do not assume a queue preserves order — most don't across partitions/shards, and redelivery can reorder even within one.
- Do not assume a comment or variable name describes current behavior — code drifts, comments don't.
- Do not assume "single writer" without finding every code path that can reach the write.
- Do not assume a platform feature ("cancel-in-progress," "exactly-once," "strongly consistent") behaves as named without checking that platform's actual documented semantics.

## Evidence Hierarchy

When evidence conflicts, higher wins. Use this order to both label a finding
and resolve disagreements between sources:

1. Runtime behavior (observed logs, traces, or read-only live inspection)
2. Tests (what's actually asserted, not just what's covered)
3. Production configuration (what's actually deployed, not the repo default)
4. Source code (the real logic path, not the interface/type signature)
5. Documentation
6. Comments
7. Assumption (no evidence found — must be labeled Unconfirmed)

Label every conclusion:

- **Verified** — confirmed at evidence tier 1-3, or tier 4 when tiers 1-3 don't apply.
- **Inferred** — supported by tier 4-6 evidence but not directly observed running.
- **Unconfirmed** — tier 7 only; requires a decision, credential, environment access, or missing source.

Do not present Inferred or Unconfirmed conclusions as fact. Browse official documentation when platform behavior may have changed. Use read-only inspection unless the user explicitly authorizes mutation.

## Review Output

Lead with the decision and the next required action. Then report:

1. Preserved invariants.
2. Regressed or unproven invariants.
3. Event and ownership flow.
4. Recognized architecture/pattern, if any, and where it diverges from that pattern's guarantees.
5. System smells found.
6. Trade-off ledger and recommended alternative.
7. Open questions — for anything Unconfirmed, ask the specific question that would resolve it (e.g. "Where is this event originally created?", "What retries this workflow, and with what identity?", "What guarantees ordering here?", "Who owns this state?", "What happens if two requests race?") instead of guessing.
8. Minimal safe change — the smallest modification that resolves the blocking findings without introducing a new mechanism, stated concretely enough to implement.

For each finding, include:

- **Severity** — Critical (violates an invariant), High (ownership leak or missing failure handling), Medium (duplicated orchestration or unmanaged trade-off), Low (readability/style, and only after 1-6 above are covered).
- **Confidence** — High/Medium/Low, stated as *why*: which evidence tier it's grounded in and what's missing (e.g. "Medium — verified in code, no test covers the concurrent case, runtime behavior unobserved"). Do not state confidence as a bare invented percentage; ground it in the evidence tier.
- The concrete failure scenario and affected component, explained before any proposed code.

Distinguish blockers (Critical/High) from hardening and optional follow-up (Medium/Low).

When reviewing a proposed fix, explicitly say whether it:

- Solves the stated problem.
- Preserves the existing system contract.
- Introduces a new collision or failure domain.
- Matches established repository architecture.
- Is the smallest defensible change.

Do not implement until any behavior-changing trade-off is either already specified or clearly accepted by the user.
