# Known Distributed-System Patterns

Load this file during Pass 2-4 when the change touches cross-boundary state,
async processing, or coordination. Naming the pattern surfaces the guarantees
it's *supposed* to provide — review the change against those guarantees, not
just against its own stated intent.

For each, check the "violated when" line against the actual diff.

- **Saga** — a sequence of local transactions with compensating actions on
  failure. Violated when a compensating step is missing, not idempotent, or
  can run after the forward step has already been compensated once (double
  compensation).
- **Outbox** — writes state and the event-to-publish in the same local
  transaction, then relays it. Violated when the event is published before
  or outside the state-changing transaction (the classic dual-write bug).
- **Inbox** — deduplicates incoming events by a stored identity before
  processing. Violated when the dedup key isn't the producer's true identity
  (e.g. a retried request gets a new key) or the inbox record isn't written
  atomically with the side effect it guards.
- **CQRS** — separate read and write models. Violated when the write path
  starts reading from the read model for correctness-critical decisions, or
  the read model's staleness window is unbounded/unmonitored.
- **Event Sourcing** — state is derived by replaying an append-only event
  log. Violated when current state is mutated directly without an event, or
  replay is not deterministic (depends on wall-clock time, external calls).
- **Leader Election** — exactly one active coordinator at a time. Violated
  when there's no fencing token, so a stale leader can still act after
  losing leadership (split-brain).
- **Reconciliation loop** — repeatedly diffs desired vs. observed state and
  converges. Violated when the loop assumes it runs to completion instead of
  being interruptible/re-entrant at any step, or when it trusts cached
  observed-state instead of re-reading it.
- **State Machine** — explicit states and legal transitions. Violated when a
  transition is applied without checking the current state first (lost
  update) or an "impossible" state is reachable via a partial failure.
- **Work Queue** — workers pull units of work; at-least-once by default.
  Violated when handlers assume at-most-once, or visibility timeout is
  shorter than real processing time (duplicate delivery under load).
- **Idempotency Key** — a client-supplied key makes repeated requests safe
  to retry. Violated when the key's scope doesn't match the actual retry
  boundary (e.g. keyed per-request but retries regenerate the key), or the
  dedup store isn't durable across the same failure that triggers the retry.
- **Circuit Breaker** — stops calling a failing dependency after a
  threshold, fails fast instead. Violated when there's no half-open probe
  (it never recovers automatically) or the breaker is scoped too broadly
  (one bad tenant trips it for everyone).
- **Bulkhead** — isolates resources (threads, connections, quota) per
  dependency so one failure can't exhaust shared capacity. Violated when a
  supposedly isolated pool actually shares a thread pool, connection pool,
  or rate limit with something else.
- **Backpressure** — a slow consumer signals producers to slow down instead
  of buffering unboundedly. Violated when the queue/buffer in front of the
  slow consumer has no bound, no drop policy, and no signal back upstream.

If the change resembles one of these but doesn't name it, say so explicitly:
*"This resembles an Outbox implementation but violates the Outbox guarantee
because the event is published via a separate connection before commit."*
That framing is more useful than a generic "this could lose the event."
