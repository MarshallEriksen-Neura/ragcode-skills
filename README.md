# RagCode Skills

[![skills.sh](https://skills.sh/b/MarshallEriksen-Neura/ragcode-skills)](https://skills.sh/MarshallEriksen-Neura/ragcode-skills)

Reusable agent skills for using [RagCode](https://github.com/MarshallEriksen-Neura/ragcode) as a local code-intelligence and shared-memory layer.

This repository is the source bundle for two skills:

- `ragcode-context` routes code understanding, ownership, impact, related-test, flow, review, freshness, and setup questions through RagCode MCP tools first, with CLI fallback.
- `ragcode-memory` routes project-scoped shared memory work through RagCode MCP memory tools (`memory_write`, `memory_query`, `memory_list`, `memory_delete`) so decisions, preferences, and cross-agent feedback persist.

## Install

Install both RagCode skills:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills
```

Install only one skill explicitly:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --skill ragcode-context
npx skills add MarshallEriksen-Neura/ragcode-skills --skill ragcode-memory
```

Install globally instead of project-local:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --global
```

Update installed RagCode skills:

```bash
npx skills update ragcode-context ragcode-memory
```

## Prerequisites

Install and configure RagCode in the target project:

```bash
npm install -g ragcode-context-engine
ragcode init
ragcode setup-mcp --client codex
ragcode index .
```

Read-time freshness is lazy by default: RagCode refreshes stale indexed files before answering. For a resident background process, choose a service mode explicitly:

```bash
ragcode service install . --mode supervisor
# or, for the legacy always-live watcher:
ragcode service install . --mode hot
```

Without a background service, the skills still work. Missing indexes still need `ragcode index .` / `index_repo`; stale indexed files can be refreshed by read-time freshness, `ragcode refresh .`, `refresh_index` through MCP, or a foreground `ragcode watch .` process.

## Verify

List skills in this repository:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --list
```

The expected list is:

```text
ragcode-context
ragcode-memory
```

Generate a one-off prompt without installing:

```bash
npx skills use MarshallEriksen-Neura/ragcode-skills --skill ragcode-context
npx skills use MarshallEriksen-Neura/ragcode-skills --skill ragcode-memory
```

## Publishing

The `MarshallEriksen-Neura/ragcode-skills` repository should publish this bundle layout as its root: root `README.md`, sibling `ragcode-context/`, and sibling `ragcode-memory/`. Publishing only `ragcode-context/` makes `npx skills add MarshallEriksen-Neura/ragcode-skills` discover only `ragcode-context`, leaving `ragcode-memory` out of normal install and update flows.

## Files

```text
ragcode-context/
  SKILL.md                 # agent routing instructions for code intelligence
  agents/openai.yaml       # OpenAI/Codex metadata
  references/cli.md        # CLI fallback reference
  references/mcp-tools.md  # MCP tool reference

ragcode-memory/
  SKILL.md                 # agent routing instructions for shared memory
  agents/openai.yaml       # OpenAI/Codex metadata
```
