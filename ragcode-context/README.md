# RagCode Skills

[![skills.sh](https://skills.sh/b/MarshallEriksen-Neura/ragcode-skills)](https://skills.sh/MarshallEriksen-Neura/ragcode-skills)

Reusable agent skill for using [RagCode](https://github.com/MarshallEriksen-Neura/ragcode) as a local code-intelligence context layer.

The `ragcode-context` skill teaches coding agents to use RagCode before manually reading files when they need code understanding, ownership, impact analysis, related tests, request-flow tracing, or diff review. It prefers RagCode MCP tools and falls back to the `ragcode` CLI when MCP is unavailable.

The skill also preserves Chinese and code-mixed user questions: agents should pass the original intent to RagCode, add exact symbols or paths when known, and inspect query-plan/reranker diagnostics before concluding context is missing.

## Install

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills
```

Install only this skill explicitly:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --skill ragcode-context
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

Read-time freshness is lazy by default: RagCode refreshes stale indexed files before answering. For a resident background process, choose a service mode explicitly:

```bash
ragcode service install . --mode supervisor
# or, for the legacy always-live watcher:
ragcode service install . --mode hot
```

Without a background service, the skill still works. Missing indexes still need `ragcode index .` / `index_repo`; stale indexed files can be refreshed by read-time freshness, `ragcode refresh .`, `refresh_index` through MCP, or a foreground `ragcode watch .` process.

## What The Skill Does

- Routes code questions to `get_context` or `find_owner` first.
- Uses `impact_analysis`, `related_tests`, and flow tools before risky edits.
- Checks `index_status` and `watch_status` when freshness matters.
- Reads language-aware query traces, domain term categories, and reranker status when search quality matters.
- Falls back to CLI commands such as `ragcode context`, `ragcode owner`, and `ragcode tests` when MCP is not available.
- Keeps setup/configuration in the terminal; the RagCode dashboard is observation-only.

## Verify

List skills in this repository:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --list
```

Generate a one-off prompt without installing:

```bash
npx skills use MarshallEriksen-Neura/ragcode-skills --skill ragcode-context
```

## Files

```text
SKILL.md                 # agent routing instructions
agents/openai.yaml       # OpenAI/Codex metadata
references/cli.md        # CLI fallback reference
references/mcp-tools.md  # MCP tool reference
```
