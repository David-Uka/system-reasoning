# system-reasoning

An [Agent Skill](https://agentskills.io) — an open, vendor-neutral format for
extending AI agents with reusable expertise. This one is a **systems-reasoning
skill**: it reviews code, designs, migrations, infrastructure, incidents, and
performance/reliability questions **from existing behavior outward**, instead
of judging a proposed fix — or a live symptom — in isolation.

## Install

This repo *is* the skill folder — its name matches the `name` field in
`SKILL.md`, so you can drop it straight into any of the conventional skill
directories your agent scans:

```bash
# Cross-client convention (project-level, works for VS Code/Copilot, Cursor, etc.)
git clone https://github.com/David-Uka/system-reasoning .agents/skills/system-reasoning

# Cross-client convention (user-level, available in every project)
git clone https://github.com/David-Uka/system-reasoning ~/.agents/skills/system-reasoning

# Claude / Claude Code (also widely supported for pragmatic compatibility)
git clone https://github.com/David-Uka/system-reasoning ~/.claude/skills/system-reasoning
```

Or update in place with `git -C <path> pull`. Some clients (Claude.ai, Claude
API) instead want the skill uploaded through their own UI/API — see
[Using skills in Claude](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
or the [Skills API Quickstart](https://docs.claude.com/en/api/skills-guide#creating-a-skill).
Check your agent's docs from the [client list](https://agentskills.io/clients)
for its exact skills path.

Validate the format yourself with the reference tool:

```bash
skills-ref validate .
```

> Agent Skills started as an Anthropic project but is now an open standard
> (see [agentskills.io](https://agentskills.io)) implemented by 40+ AI coding
> tools — Claude, Claude Code, GitHub Copilot, VS Code, Cursor, OpenAI Codex,
> Gemini CLI, Goose, Roo Code, JetBrains Junie, Amp, OpenHands, and more (full
> list: [agentskills.io/clients](https://agentskills.io/clients)). A skill is
> just a folder with a `SKILL.md` file — any compatible agent can load it.

Trade-off auditing is the default mode and the backbone every other mode
reuses. It forces a fixed reasoning order before any change is accepted:

```
current behavior and invariants
  -> problem being solved
  -> proposed mechanism
  -> affected callers and boundaries
  -> concurrency and failure behavior
  -> trade-offs and alternatives
  -> recommendation
```

## Modes

The skill picks a mode based on what you hand it, then runs a real procedure
for that mode — not just a relabeled checklist:

| You're looking at | Mode |
|---|---|
| A PR, patch, or proposed mechanism | **Change Review** — the five passes below |
| A design doc/RFC with no diff yet | **Design Review** — same five passes, the proposal is the mechanism |
| A schema migration or a monolith→service/provider-swap migration | **Migration Review** — invariants + trade-off ledger, plus the database playbook |
| A live outage or a bug you can't explain yet | **Incident / Debugging** — timeline, hypothesis ranking, root cause vs. trigger vs. contributing factors, blast radius |
| A regression, scaling question, or SLO gap | **Performance / Reliability** — measure before guessing, classify latency/throughput/error-rate/cost, check redundancy is tested not assumed |

Every mode shares the same Evidence Hierarchy, System Smells, and Assumptions
to Never Make — that shared toolkit is what makes this a systems-reasoning
skill rather than five unrelated checklists.

## How Change/Design/Migration Review works

Five passes, run in order:

1. **Establish Invariants** — state what must keep working, as observable behavior, ranked by the Evidence Hierarchy below — never inferred from comments alone.
2. **Trace Provenance and Ownership** — follow one real event end to end (trigger → producer → transport → consumer → mutation → completion) and confirm each value's owner, validator, and source of truth. Loads the matching Kubernetes/AWS/CI/database playbook if relevant.
3. **Exercise Time and Failure** — concurrent events, retries after partial success, duplicates, mid-flight failures, malformed input. Verify actual platform semantics instead of trusting labels like "idempotent" or "atomic." Names the distributed-system pattern in play (Saga, Outbox, CQRS, Circuit Breaker, ...) and checks it against that pattern's actual guarantees.
4. **Build a Trade-off Ledger** — problem / gain / cost / cost owner / alternative / decision, for every meaningful change.
5. **Justify Every Component** — one line per new file, script, controller, CRD, workflow job, secret path, or validation block: what it does and why the flow needs it. Cut duplicated orchestration and dead compatibility branches; never cut validation that prevents false success or destructive ambiguity.

Along the way it scans for **System Smells** (hidden source of truth, circular
ownership, temporal coupling, retry amplification, fan-out explosion,
configuration drift, ...) and refuses a set of specific reasoning shortcuts
(tests passing ≠ correct, retry ≠ safe, "atomic"/"exactly-once" ≠ verified
without checking the platform's actual semantics).

Every conclusion is ranked by an explicit **Evidence Hierarchy** — runtime
behavior > tests > production config > source code > docs > comments >
assumption — and labeled **Verified**, **Inferred**, or **Unconfirmed**.
Nothing gets implemented until behavior-changing trade-offs are explicit and
accepted.

## Output format

The skill always leads with the decision and next required action, then:

1. Preserved invariants
2. Regressed or unproven invariants
3. Event and ownership flow
4. Recognized architecture/pattern and where it diverges
5. System smells found
6. Trade-off ledger and recommended alternative
7. Open questions for anything Unconfirmed — not a guess
8. Minimal safe change — the smallest fix that resolves the blockers

Each finding carries a **Severity** (Critical/High/Medium/Low, tied to
whether it violates an invariant, leaks ownership, duplicates orchestration,
or is just style) and a **Confidence** grounded in which evidence tier backs
it — never a bare invented percentage.

## Reference files

Domain and mode detail lives in `references/` so the core methodology in
`SKILL.md` stays short; the agent loads a file only when it's relevant:

- `distributed-patterns.md` — Saga, Outbox, Inbox, CQRS, Event Sourcing, Leader Election, Reconciliation, State Machine, Work Queue, Idempotency Key, Circuit Breaker, Bulkhead, Backpressure, each with a concrete "violated when."
- `domain-kubernetes.md`, `domain-aws.md`, `domain-ci-cd.md`, `domain-databases.md` — the failure modes generic review misses in each stack.
- `incident-debugging.md` — the procedure for a live/reported problem instead of a diff.
- `performance-reliability.md` — the procedure for "is this fast/reliable enough."

## Spec

Built to the [Agent Skills specification](https://agentskills.io/specification):
YAML frontmatter (`name`, `description`, `license`, `metadata`) + a Markdown
body + a `references/` folder for progressive disclosure — loaded only when
the relevant mode or domain applies, so the core methodology stays under the
spec's recommended context budget.

## License

MIT — see [LICENSE](./LICENSE).
