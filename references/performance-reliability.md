# Mode: Performance and Reliability Reasoning

Use when the question is "is this fast/reliable enough" or "will this scale/
survive a failure," not "does this change preserve behavior." Reuses this
skill's Trade-off Ledger, System Smells, and Evidence Hierarchy.

## Procedure

1. **Define the target precisely before reasoning about it.** Which metric
   (p50/p95/p99 latency, throughput, error rate, availability), whose SLO or
   budget, target vs. observed. Refuse to reason about "slow" or "unreliable"
   without a number — that number is what makes a fix falsifiable.
2. **Measure before guessing the bottleneck.** A profile, trace, or metric
   (tier-1 evidence) beats intuition about which path is hot. Treating an
   assumed hot path as the culprit without measuring it is the single most
   common wasted-effort mistake in this mode.
3. **Classify the problem** — latency, throughput, error rate, and cost are
   different axes with sometimes-opposing fixes (batching improves
   throughput but can hurt p99 latency; caching improves latency but adds a
   staleness/invalidation problem).
4. **Check the well-known culprits before anything exotic:** N+1 queries,
   missing indexes/full scans, unbounded fan-out, synchronous calls that
   could be async/batched, retry amplification, lock contention,
   connection-pool or GC exhaustion, cold starts. These overlap with this
   skill's System Smells list — use it.
5. **Reliability specifically:** identify the actual failure domain and
   redundancy. A second instance/AZ/replica that has never taken over in a
   real or drilled failover is not redundancy, it's an assumption. Confirm
   an error budget is actually being breached before spending effort — 
   hardening a system that already meets its SLO is over-engineering and
   belongs in the Trade-off Ledger as a cost, not a free win.
6. **Run any proposed fix through the Trade-off Ledger.** A performance fix
   that trades correctness, simplicity, or an existing invariant for speed
   needs explicit sign-off, same as any other trade-off in this skill.
7. **Confirm with a repeatable before/after measurement.** "Should be
   faster" is Inferred at best; a rerun benchmark or production metric is
   Verified.

## Anti-patterns specific to this mode

- Do not assume caching is free — it is a Hidden Source of Truth (System
  Smell) with a staleness/invalidation problem attached.
- Do not assume horizontal scaling fixes a bottleneck caused by lock
  contention or a shared downstream dependency — it can make it worse
  (thundering herd, connection exhaustion).
- Do not assume redundancy provides reliability without a tested failover;
  untested failover is unverified, not Verified.
- Do not accept "should be more reliable" without stating which failure
  mode it addresses and which SLO or budget justifies the cost.

## Output shape

Target and current measurement -> Verified bottleneck (with evidence tier)
-> Classification (latency/throughput/error-rate/cost) -> Trade-off Ledger
for the proposed fix -> Minimal safe change -> Confirmation method (how the
fix will be measured, not assumed).
