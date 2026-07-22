# audit-system-tradeoffs

A [Claude Agent Skill](https://support.claude.com/en/articles/12512176-what-are-skills) that reviews code, workflows, infrastructure, and distributed-system changes **from existing behavior outward** — instead of judging a proposed fix in isolation.

The skill forces a fixed reasoning order before any change is accepted:

```
current behavior and invariants
  -> problem being solved
  -> proposed mechanism
  -> affected callers and boundaries
  -> concurrency and failure behavior
  -> trade-offs and alternatives
  -> recommendation
```

## When to use it

Ask Claude to use this skill when reviewing a PR, design doc, or infra/workflow change and you want it checked for:

- preserved invariants (not just "does the new code work")
- event and input provenance across boundaries
- concurrency, retries, partial failure, and duplicate/stale events
- ownership boundaries (logic living in the wrong component)
- unnecessary or redundant logic that duplicates what's already owned elsewhere
- silent, uncommunicated trade-offs

## How it works

Five passes, run in order:

1. **Establish Invariants** — state what must keep working, as observable behavior, sourced from code/config/tests/docs — never inferred from comments alone.
2. **Trace Provenance and Ownership** — follow one real event end to end (trigger → producer → transport → consumer → mutation → completion) and confirm each value's owner, validator, and source of truth.
3. **Exercise Time and Failure** — concurrent events, retries after partial success, duplicates, mid-flight failures, malformed input. Verify actual platform semantics instead of trusting labels like "idempotent" or "atomic."
4. **Build a Trade-off Ledger** — problem / gain / cost / cost owner / alternative / decision, for every meaningful change.
5. **Justify Every Component** — one line per new file, script, controller, CRD, workflow job, secret path, or validation block: what it does and why the flow needs it. Cut duplicated orchestration and dead compatibility branches; never cut validation that prevents false success or destructive ambiguity.

Every conclusion is labeled **Verified**, **Inferred**, or **Unconfirmed**. Nothing gets implemented until behavior-changing trade-offs are explicit and accepted.

## Output format

The skill always leads with the decision and next required action, then:

1. Preserved invariants
2. Regressed or unproven invariants
3. Event and ownership flow
4. Trade-off ledger and recommended alternative
5. Unconfirmed decisions that block implementation

## Install

**Claude.ai / Claude Code / Claude API** — see [Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude) and the [Skills API Quickstart](https://docs.claude.com/en/api/skills-guide#creating-a-skill). Drop this folder (or its `SKILL.md`) in wherever your setup loads custom skills from.

## License

MIT — see [LICENSE](./LICENSE).
