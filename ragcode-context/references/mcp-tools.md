# RagCode MCP Tools Reference (for agents)

Server name: `ragcode` (stdio). All tools accept `repoRoot` (optional when a workspace default is resolvable) and return JSON.

## Indexing and state

- `index_repo` `{ repoRoot }` — build/refresh the index. Run this first on a new repo.
- `refresh_index` `{ repoRoot? }` — incremental refresh of dirty files.
- `index_status` `{ repoRoot?, full? }` — light freshness/dirty status by default; pass `full: true` only when chunk, symbol, edge, and semantic-store counts are needed.
- `watch_status` `{ repoRoot?, full? }` — read-only watcher/supervisor liveness and backlog. It never starts a watcher.

## Primary retrieval tools (prefer these)

- `get_context` `{ repoRoot?, query, mode?, budgetChars?, limit?, format? }` — agent-ready context pack with snippets, topology, coverage, query-plan diagnostics, and optional `format: "json" | "markdown"`. Modes: `debug`, `feature`, `refactor`, `review`, `explain`.
- `find_owner` `{ repoRoot?, query, limit? }` — likely owner files/symbols for a behavior.
- `impact_analysis` `{ repoRoot?, target }` — structural blast radius for a file or symbol.
- `related_tests` `{ repoRoot?, target }` — tests covering a target.
- `trace_flow` `{ repoRoot?, entry, maxSteps? }` — request/data flow from an entry point.
- `review_diff` `{ repoRoot?, diff? | changedFiles? }` — review evidence for a change.

## Secondary tools

- `search_code` `{ repoRoot?, query, limit?, mode? }` — raw hybrid search hits.
- `find_symbol` `{ repoRoot?, name }` — exact symbol lookup.
- `explain_file` `{ repoRoot?, filePath }` — file card with chunks and symbols.
- `find_reuse_candidates` `{ repoRoot?, query, limit?, reuseGuard? }` — existing implementations to reuse before writing new code; `reuseGuard: true` hard-blocks confirmed duplicates.
- `trace_request_flow` / `explain_impact` / `verified_subgraph` / `expand_node` / `topology_map` — verified graph evidence and node expansion under budget.

## Recommended flows

- Code change: `get_context` → edit → `related_tests` → `review_diff`.
- "Where do I fix X?": `find_owner` → `get_context` on the owner.
- Risky refactor: `impact_analysis` → `verified_subgraph` (mode `impact`) → `related_tests`.
- Chinese/code-mixed query: preserve the user query, add exact symbols/paths if known, call `get_context`, then inspect `trace.queryPlan.detectedLanguage`, `termCategories`, and weighted match reasons.
- Reranker concern: inspect `diagnostics.reranker.status`, `provider`, `candidateCount`, `scoredCount`, and `error`; failed external reranker calls should still have graph-reranked fallback results.
- Stale/no data: `index_status` → `refresh_index` for stale indexed repos or `index_repo` for missing indexes → retry original tool. v0.1.11 read paths also refresh a bounded dirty set on demand before answering.
- Auto-refresh concern: `watch_status`; if not running, remember lazy read-time refresh may still be enough. Use/suggest `ragcode service install <repoRoot> --mode supervisor` for a light resident event recorder, `--mode hot` for the legacy always-live watcher, or `ragcode watch <repoRoot>` for a foreground watcher.
