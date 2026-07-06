# RagCode CLI Reference (for agents)

Read commands require an existing index, then refresh stale indexed files on demand by default. Reads resolve runtime config via CLI args > env > `<repoRoot>/.ragcode/config.json` > offline-first defaults (sqlite + lancedb + deterministic). User-facing `ragcode configure --show` / `--test` resolve the saved repo config first and redact secrets.

## Health and state

```bash
ragcode doctor [repoRoot] --query "<smoke query>"  # deps, runtime config (redacted), MCP registration, optional index/search smoke
ragcode status <repoRoot>                          # light persisted index + dirty watcher state, no heavy store load
ragcode status <repoRoot> --full                   # full graph/semantic/symbol/chunk status
ragcode status-human <repoRoot>                    # human-readable watch/index/embedding status
ragcode status-ui <repoRoot>                       # alias for status-human
ragcode service status <repoRoot>                  # background watcher service/liveness status
```

## Indexing

```bash
ragcode index <repoRoot>          # full/incremental index
ragcode refresh <repoRoot>        # refresh an already-indexed repo
ragcode index <repoRoot> [--max-batch-files N] [--max-analysis-memory-mb N]
ragcode index <repoRoot> --semantic-on-bootstrap  # also write vectors for first partial bootstrap batch
ragcode index <repoRoot> --full                   # force legacy all-at-once index
ragcode watch <repoRoot>          # long-lived watcher daemon with background refresh
ragcode watch <repoRoot> [--max-batch-files N] [--max-analysis-memory-mb N]
ragcode service install <repoRoot>                 # lazy mode: installs no resident watcher
ragcode service install <repoRoot> --mode supervisor [--index-now] [--bootstrap-batch-size N]
ragcode service install <repoRoot> --mode hot [--index-now] [--bootstrap-batch-size N]
ragcode service uninstall <repoRoot>
```

Empty indexes use a bounded bootstrap batch by default. Remaining files are persisted as pending dirty state and progress is written to `.ragcode/index-state.json` / `.ragcode/index-progress.jsonl`. `service install` does not block on a full index unless `--index-now` is provided.

## Retrieval

```bash
ragcode context <repoRoot> "<query>" [--mode debug|feature|refactor|review|explain] [--budget <chars>]
ragcode search <repoRoot> "<query>" [--limit N]
ragcode owner <repoRoot> "<query>"
ragcode reuse <repoRoot> "<query>"          # reusable existing code before writing new code
ragcode impact <repoRoot> <fileOrSymbol>
ragcode explain-impact <repoRoot> <target>  # verified minimal subgraph for blast radius
ragcode tests <repoRoot> <fileOrSymbol>
ragcode trace-request-flow <repoRoot> <entry>
ragcode expand-node <repoRoot> <file[:symbol]> [--expansion focused_body|full_body|skeleton|file_card]
```

Retrieval/read commands accept `--stale-ok` / `--no-refresh` to return existing index results without on-demand refresh, `--refresh` to force refresh, and `--max-analysis-memory-mb N` to bound refresh memory. `RAGCODE_REFRESH_ON_READ=off|stale-ok|no-refresh` disables read-time refresh globally; `force|force-refresh|always` forces it.

## Setup and configuration

```bash
ragcode init [dir] [--defaults]      # first-run config; --defaults writes offline-first config without prompts
ragcode configure [repoRoot]         # edit storage/embedding config; --show prints effective config; --test verifies embedding
ragcode setup-mcp [--print] [--include-secrets] [--force] [--client claude|claude-code|codex|generic]
ragcode mcp                          # start the MCP server over stdio
ragcode update [--check]              # update globally-installed RagCode CLI, or check latest
ragcode dashboard                    # Web observability API (humans only)
```

External reranking is configured separately from embeddings through env/config keys, not a separate CLI command:

```bash
export RAGCODE_RERANK_PROVIDER=openai-compatible
export RAGCODE_RERANK_BASE_URL=https://your-router.example/v1
export RAGCODE_RERANK_API_KEY=your-key
export RAGCODE_RERANK_MODEL=your-rerank-model
export RAGCODE_RERANK_PATH=/rerank
export RAGCODE_RERANK_TOP_N=80
```

`RAGCODE_RRANK_*` aliases are accepted. `ragcode configure --show` and diagnostics redact secrets.

## Error recovery

- "Repository is not indexed" → run `ragcode index <repoRoot>`, retry.
- Embedding failures → `ragcode configure --test` to classify (missing key / auth / model / network / dimensions), then `ragcode configure` to fix.
