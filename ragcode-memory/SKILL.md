---
name: ragcode-memory
description: Manage shared cross-AI memories for this project through RagCode MCP tools (memory_write, memory_query, memory_list, memory_delete), with ragcode memory CLI fallback when MCP is unavailable. Read prior decisions, user preferences, and feedback written by other AI agents, and record new insights so they persist across sessions and agents. Trigger when context involves past decisions, user preferences, architectural choices, corrections, or when switching between AI agents (Claude, Codex, Cursor, JoyCode).
---

# RagCode Memory

RagCode provides a **shared, project-scoped memory** that persists across sessions
and is visible to every AI agent connected to the same repo over MCP. What one
agent records, another agent reads. Memory is bound to the repo root, not to any
agent — it does not leak across projects.

## When to use

- **Session start / agent switch** → `memory_list` (or `get_context`, which now
  attaches `memoryHints`) to load key decisions and preferences before assuming a
  blank slate.
- **User states a preference, rule, or constraint** → `memory_write` (type `user`
  or `feedback`).
- **A decision or architectural choice is made** → `memory_write` (type
  `decision`).
- **User corrects your approach** → `memory_write` (type `feedback`).
- **Before a non-trivial change** → `memory_query` for related prior decisions.
- **A tool response includes `memoryHints` or `memorySnippets`** → follow up with
  `memory_query` to explore.

## MCP memory tools (server: `ragcode`)

| Action | Tool | When |
|---|---|---|
| Record a memory | `memory_write` | preference stated, decision made, lesson learned |
| Search memories | `memory_query` | before risky changes, when hints appear, checking past decisions |
| List / browse | `memory_list` | session start, agent switch, catching up (chronological) |
| Remove | `memory_delete` | memory is wrong (prefer `supersede` on write instead) |

## CLI fallback (MCP unavailable)

```bash
ragcode memory write <topic> --type <type> --title "<title>" --body "<markdown>"
ragcode memory query "<query>" [--mode exact|semantic|hybrid] [--limit <n>]
ragcode memory list [--type <type>] [--topic <topic>] [--limit <n>]
ragcode memory delete <id> --reason "<reason>"
```

Prefer MCP tools when available because they preserve structured results for the calling agent. Use the CLI fallback for local diagnostics, recovery, or non-MCP agent surfaces.

## Memory types

| Type | Write when | Example |
|---|---|---|
| `user` | user preference / trait | "prefers Go over Rust" |
| `feedback` | user correction / rule | "don't mock the database in tests" |
| `project` | project state / fact | "auth middleware rewritten for compliance" |
| `reference` | external pointer | "bugs tracked in Linear INGEST" |
| `decision` | architectural / design choice | "chose event sourcing over CRDT" |
| `context` | current work context | "refactoring the payment module" |

## Search modes

`memory_query` accepts `mode`: `exact` (keyword FTS), `semantic` (embedding
similarity), or `hybrid` (both, RRF-fused — the default). Semantic/hybrid require
the server to have embeddings enabled (`RAGCODE_MEMORY_SEMANTIC=on`); otherwise
they transparently fall back to keyword search. Results are ranked by MQS_v3
(quality), so the most useful memory surfaces, not merely the newest.

## Writing guidelines

- **Be specific**: "use SQLite in WAL mode" > "use a database".
- **Include why**: add a **Why:** line.
- **Include how to apply**: add a **How to apply:** line for future context.
- **Set confidence**: 0.7–1.0 for verified facts, 0.3–0.6 for assumptions.
- **Prefer supersede**: when updating a memory, pass `supersede` with the old id to
  keep the audit chain instead of deleting.
- **Heed the write result**: `memory_write` returns `duplicates` (existing
  near-identical memories — consider `supersede` instead of piling on) and
  `conflicts` (same-topic memories from another agent that may disagree — review
  and reconcile).
- **Never store secrets**: API keys, passwords, and tokens must NOT be memorized.

## Lifecycle (automatic — you don't manage this)

- Memories are **verified** against the codebase when the repo is re-indexed;
  stale/invalid ones are marked and de-prioritized.
- Low-value and long-idle memories are **archived** automatically by the
  maintenance pass; near-duplicates are **merged** (superseded to the best copy).
- **Injection**: `get_context` attaches the most relevant memories as
  `memorySnippets` (MAB-routed) plus a `memoryHints` summary.

## Integration with other RagCode tools

- On session start: `memory_list` → `get_context` (memories inform the query).
- Before editing: `memory_query` for related decisions → `get_context` → edit.
- After a user correction: `memory_write(type=feedback)` → continue.
- After an architectural decision: `memory_write(type=decision)` → continue.
- When `get_context` returns `memoryHints.conflicts` → `memory_query` those, then
  `memory_write(supersede=…)` to reconcile.
