# audit-system-tradeoffs

An [Agent Skill](https://agentskills.io) — an open, vendor-neutral format for
extending AI agents with reusable expertise. This one reviews code, workflows,
infrastructure, and distributed-system changes **from existing behavior
outward**, instead of judging a proposed fix in isolation.

> Agent Skills started as an Anthropic project but is now an open standard
> (see [agentskills.io](https://agentskills.io)) implemented by 40+ AI coding
> tools — Claude, Claude Code, GitHub Copilot, VS Code, Cursor, OpenAI Codex,
> Gemini CLI, Goose, Roo Code, JetBrains Junie, Amp, OpenHands, and more (full
> list: [agentskills.io/clients](https://agentskills.io/clients)). A skill is
> just a folder with a `SKILL.md` file — any compatible agent can load it.

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

Ask your agent to use this skill when reviewing a PR, design doc, or
infra/workflow change and you want it checked for:

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

This repo *is* the skill folder — its name matches the `name` field in
`SKILL.md`, so you can drop it straight into any of the conventional skill
directories your agent scans:

```bash
# Cross-client convention (project-level, works for VS Code/Copilot, Cursor, etc.)
git clone https://github.com/David-Uka/audit-system-tradeoffs .agents/skills/audit-system-tradeoffs

# Cross-client convention (user-level, available in every project)
git clone https://github.com/David-Uka/audit-system-tradeoffs ~/.agents/skills/audit-system-tradeoffs

# Claude / Claude Code (also widely supported for pragmatic compatibility)
git clone https://github.com/David-Uka/audit-system-tradeoffs ~/.claude/skills/audit-system-tradeoffs
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

## Spec

Built to the [Agent Skills specification](https://agentskills.io/specification):
YAML frontmatter (`name`, `description`, `license`, `metadata`) + a Markdown
body, no scripts or bundled resources required for this one.

## License

MIT — see [LICENSE](./LICENSE).
