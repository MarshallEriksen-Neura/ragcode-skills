# RagCode Memory Skill

Reusable agent skill for using [RagCode](https://github.com/MarshallEriksen-Neura/ragcode) as a project-scoped shared memory layer.

The `ragcode-memory` skill teaches agents to read prior decisions, user preferences, architectural choices, corrections, and cross-agent feedback before acting, then record new durable insights through RagCode MCP memory tools.

This skill is published together with `ragcode-context` in the `MarshallEriksen-Neura/ragcode-skills` bundle. Default `npx skills add MarshallEriksen-Neura/ragcode-skills` installs both skills; use `--skill ragcode-memory` only when you want this skill alone.

## Install

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills
```

Install only this skill explicitly:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --skill ragcode-memory
```

Install globally instead of project-local:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --global
```

## Prerequisites

Install and configure RagCode in the target project:

```bash
npm install -g ragcode-context-engine
ragcode init
ragcode setup-mcp --client codex
ragcode index .
```

The memory skill requires the RagCode MCP server because it calls `memory_write`, `memory_query`, `memory_list`, and `memory_delete`.

## What The Skill Does

- Reads project-scoped memories before non-trivial work.
- Searches prior decisions and user preferences with `memory_query`.
- Lists recent memory state with `memory_list` during session starts or agent handoffs.
- Records new durable preferences, decisions, corrections, and lessons with `memory_write`.
- Avoids storing secrets and prefers superseding outdated memories over deleting them.
- Pairs with `ragcode-context` so remembered decisions can guide fresh code ownership, impact, and test lookups.

## Verify

List skills in this repository:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --list
```

Generate a one-off prompt without installing:

```bash
npx skills use MarshallEriksen-Neura/ragcode-skills --skill ragcode-memory
```

## Files

```text
SKILL.md           # agent routing instructions for shared memory
agents/openai.yaml # OpenAI/Codex metadata
```
