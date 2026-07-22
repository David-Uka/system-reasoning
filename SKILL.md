---
name: audit-system-tradeoffs
description: Review code, workflows, infrastructure, and distributed-system changes from existing behavior outward. Use when evaluating a PR or design for preserved invariants, hidden trade-offs, event and input provenance, concurrency, retries, partial failure, ownership boundaries, unnecessary logic, or system-level regressions caused by a locally correct fix.
license: MIT
metadata:
  author: David Uka
  version: "1.0"
---

# Audit System Trade-offs

Review the system before judging the proposed mechanism. Establish what must remain true, trace real events across boundaries, and make every behavioral trade-off explicit.

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

## Pass 1: Establish Invariants

Read the existing implementation and identify behaviors that users or dependent systems rely on.

Ask:

- What works today and must continue working?
- Which events, updates, or records must never be lost?
- Which ordering, immutability, security, and human-review guarantees exist?
- Which behavior is intentionally manual or automatic?
- Which shared paths have callers outside this feature?

Write the invariants as observable statements, not implementation details. For example: Every accepted service update eventually reaches the platform chart is an invariant; Use a concurrency group is a mechanism.

Do not infer an invariant solely from comments. Confirm it from code, configuration, documentation, tests, or live read-only evidence when available.

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

## Evidence Discipline

Label conclusions as:

- Verified: Confirmed from source, tests, official documentation, or read-only live state.
- Inferred: Strongly supported but not directly observed.
- Unconfirmed: Requires a decision, credential, environment access, or missing source.

Do not present inferred or unconfirmed behavior as fact. Browse official documentation when platform behavior may have changed. Use read-only inspection unless the user explicitly authorizes mutation.

## Review Output

Lead with the decision and the next required action. Then report:

1. Preserved invariants.
2. Regressed or unproven invariants.
3. Event and ownership flow.
4. Trade-off ledger and recommended alternative.
5. Unconfirmed decisions that genuinely block implementation.

For findings, include the concrete failure scenario and affected component. Explain why the issue matters before proposing code. Distinguish blockers from hardening and optional follow-up work.

When reviewing a proposed fix, explicitly say whether it:

- Solves the stated problem.
- Preserves the existing system contract.
- Introduces a new collision or failure domain.
- Matches established repository architecture.
- Is the smallest defensible change.

Do not implement until any behavior-changing trade-off is either already specified or clearly accepted by the user.
