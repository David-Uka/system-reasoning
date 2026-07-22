# system-reasoning

A systems-reasoning [Agent Skill](https://agentskills.io) for Claude, Cursor,
Copilot, Codex, and 40+ other AI coding tools. Reviews code, designs,
migrations, incidents, and performance questions **from existing system
behavior outward** — instead of judging a proposed fix in isolation.

## Install

This repo *is* the skill folder — drop it straight into any of the
conventional skill directories your agent scans:

```bash
# Cross-client (project-level)
git clone https://github.com/David-Uka/system-reasoning .agents/skills/system-reasoning

# Cross-client (user-level, all projects)
git clone https://github.com/David-Uka/system-reasoning ~/.agents/skills/system-reasoning

# Claude / Claude Code
git clone https://github.com/David-Uka/system-reasoning ~/.claude/skills/system-reasoning
```

Update in place with `git -C <path> pull`. Claude.ai / Claude API users:
upload it through the [Claude skills UI](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
instead. Check your own agent's docs from the [client list](https://agentskills.io/clients)
for its exact skills path.

Validate the format:

```bash
skills-ref validate .
```

## What it does

| You hand it | It runs |
|---|---|
| A PR or patch | **Change Review** — invariants → provenance → failure modes → trade-off ledger → recommendation |
| A design doc / RFC | **Design Review** — same five passes, proposal as the mechanism |
| A schema or service migration | **Migration Review** — invariants + trade-off ledger + database playbook |
| A live outage or unexplained bug | **Incident / Debugging** — timeline, hypothesis ranking, blast radius |
| A regression or SLO gap | **Performance / Reliability** — measure before guessing |

Every mode shares one toolkit: an **Evidence Hierarchy** (runtime behavior >
tests > config > source > docs > comments > assumption), a set of **System
Smells** it scans for (hidden source of truth, temporal coupling, retry
amplification, config drift...), and reasoning shortcuts it refuses to take
(tests passing ≠ correct, retry ≠ safe, "atomic" ≠ verified).

Output always leads with the decision and next required action, then the
evidence behind it — every finding gets a Severity, a Confidence, and is
labeled Verified / Inferred / Unconfirmed. Full methodology lives in
[`SKILL.md`](./SKILL.md).

## Reference files

Domain and mode detail lives in `references/`, loaded only when relevant, so
`SKILL.md` stays short:

- `distributed-patterns.md` — Saga, Outbox, CQRS, Circuit Breaker, and 9 more, each with a "violated when"
- `domain-kubernetes.md`, `domain-aws.md`, `domain-ci-cd.md`, `domain-databases.md` — stack-specific failure modes
- `incident-debugging.md`, `performance-reliability.md` — the non-diff modes

## Spec

Built to the [Agent Skills specification](https://agentskills.io/specification):
YAML frontmatter + Markdown body + a `references/` folder for progressive
disclosure, kept under the spec's recommended context budget.

## License

MIT — see [LICENSE](./LICENSE).
