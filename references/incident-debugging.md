# Mode: Incident Analysis and Production Debugging

Use when investigating a live or reported problem, not reviewing a proposed
change. Reuses this skill's Evidence Hierarchy, System Smells, and
Assumptions-to-Never-Make — the difference is the procedure that gets you to
a claim worth labeling Verified/Inferred/Unconfirmed in the first place.

## Procedure

1. **Pin the symptom precisely.** What's actually observed — which metric,
   how much deviation, since when, affecting whom. "It's broken" is not a
   symptom; "p99 checkout latency went from 200ms to 4s at 14:02 UTC for
   EU traffic only" is.
2. **Reconstruct the timeline.** Last known good state, first bad signal,
   and every deploy/config/traffic/dependency change in that window.
   Correlation in time is a lead, not a conclusion.
3. **Trace the actual failing path**, reusing Pass 2's trigger -> producer
   -> transport -> consumer -> mutation -> completion trace. Find exactly
   where observed behavior diverges from the traced path.
4. **Generate multiple hypotheses before picking one.** Rank candidate
   causes by the Evidence Hierarchy (runtime/tests/config/code above
   docs/comments/assumption). Do not stop at the first hypothesis that
   fits the timeline — falsify it against tier-1 evidence before committing.
5. **Separate trigger, root cause, and contributing factors.** A deploy can
   be the trigger while a missing invariant (e.g. no idempotency check) is
   the root cause; a third system's slow retries can be a contributing
   factor. Report all three, not just whichever is easiest to point at.
6. **Determine blast radius.** What else shares this failure domain and may
   be silently affected right now, not just the component that alerted.
7. **Mitigate before fully diagnosing, unless mitigation destroys the
   evidence needed to diagnose.** If restarting, rolling back, or purging a
   queue would erase the one artifact that explains root cause (a poison
   message, a corrupt record), capture it first.
8. **Write corrective actions with an owner and a mechanism**, not just a
   monitor. Ask whether the invariant that broke should be enforced
   structurally (a constraint, a type, a check) rather than watched for.
9. **Stay blameless.** Describe the system gap, not a person's mistake.

## Anti-patterns specific to this mode

- Do not treat two concurrent bad signals as related without evidence they
  share a cause.
- Do not stop at the first plausible story — falsify it with tier-1
  evidence before writing it down as the cause.
- "It went away after we restarted X" is not root cause — a restart clears
  the symptoms of a memory leak, a stuck lock, and a poison message equally
  well. State what specifically changed, not that something changed.
- Do not let mitigation erase the evidence needed to prevent recurrence.

## Output shape

Timeline -> Trigger vs. Root Cause vs. Contributing Factors -> Blast Radius
-> Evidence-ranked findings (Verified/Inferred/Unconfirmed, per this
skill's Evidence Hierarchy) -> Immediate mitigation -> Corrective actions
(owner + mechanism) -> Open questions for anything Unconfirmed.
