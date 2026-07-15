---
name: ragcode-agents
description: Orchestrate multi-agent workflows through RagCode. Turn a natural-language collaboration pattern (e.g. "Gemini plans, Claude breaks down, Codex implements, Claude reviews") into a validated DAG workflow spec, then run it with the RagCode agent workflow engine. Trigger when the user asks to coordinate multiple AI agents, run a plan-implement-review loop, or orchestrate codex/claude/gemini/grok together on one task.
---

# RagCode Agents

RagCode orchestrates multiple AI agents (codex, claude, gemini, grok, or any CLI
configured in `.ragcode/agents/agents.json`) through a **validated DAG workflow**.
A user describes a collaboration pattern in natural language; you generate a
workflow spec as **JSON data**; RagCode validates and executes it with review
gates, bounded fix loops, and an append-only run ledger.

This is NOT a fixed "claude + codex parallel consult" command. The architecture
supports arbitrary role-to-agent assignment, serial and parallel stages,
review/fix loops, and explicit write permissions.

## When to use

- **User describes a multi-agent collaboration** → generate a workflow spec.
- **User wants plan → implement → review** with different agents → workflow with a review gate.
- **User names specific agents for specific roles** ("let Gemini plan, Codex implement") → preserve their assignment.
- **User wants to run a previously-defined workflow** → `ragcode agents run`.

## Core rules (follow strictly)

1. **Preserve the user's explicit role-to-provider assignment.** If they say "Gemini plans, Codex implements", do not swap to "Claude plans, Codex implements".
2. **Generate workflow JSON, not prose.** The spec is data; RagCode validates it.
3. **Read-only stages** (`plan`, `breakdown`, `review`, `verify`) use `"mode": "read_only"`.
4. **Implementation stages** (`implement`, `fix`) use `"mode": "write"` and MUST declare `writePolicy` with `allowedPaths`.
5. **Add a review gate** with `onFail` + `maxLoops` (1..10) whenever a reviewer is named.
6. **Always validate before run**: `ragcode agents validate . --workflow <path>`.
7. **Prefer `--dry-run` first** unless the user explicitly asked to execute.
8. **Use presets only when the user did not specify roles.**

## Workflow spec (v1)

A spec is JSON with `version`, `name`, `description`, `agents` (role → provider mapping), `workflow` (ordered DAG nodes), and optional `finalGate`. Each node declares `id`, `agent`, `kind`, `mode`, `dependsOn`, `input`/`output`, and optional `gate` or `writePolicy`.

Read `references/workflow-schema.md` for the full schema, a complete example, all validation rules, and preset definitions.

## Presets

Use these only when the user did NOT specify roles:

- **`consult`** — parallel read-only analysis, no writes.
- **`plan-execute-review`** — planner → executor → reviewer with one review gate.
- **`gemini-claude-codex-review`** — Gemini plans, Claude breaks down, Codex implements, Claude reviews with a fix loop.

## CLI commands

```bash
ragcode agents init .                                                    # create .ragcode/agents/
ragcode agents validate . --workflow .ragcode/agents/workflows/feature.json
ragcode agents run . --workflow .ragcode/agents/workflows/feature.json --task "Implement X" --dry-run
ragcode agents run . --workflow .ragcode/agents/workflows/feature.json --task "Implement X"
ragcode agents status . --run <runId>
ragcode agents report . --run <runId>
ragcode agents resume . --run <runId> --workflow <path> --task "<task>"
ragcode agents cancel . --run <runId>
ragcode agents list .                                                   # list durable run ids
ragcode agents start . --workflow <path> --task "<task>"                # detached run
ragcode agents events . --run <runId> --no-follow                       # ledger event stream snapshot
ragcode agents invoke . --run <runId> --message "<message>"             # mid-turn message to active node
```

`--json` is available on most commands. Read `references/cli.md` for the full command reference and error recovery.

## Safety model

- **One workspace write node at a time** (V1 conservative mode). V2 will use git worktrees.
- **Write nodes are confined to `writePolicy.allowedPaths`** (glob patterns, repo-relative).
- **Review failures stop fail-closed** after `maxLoops` — the run does not continue with unverified output.
- **Missing CLI fails before run start** with an actionable diagnostic, not a partial run.
- **Secrets are redacted** from the run ledger.
- **MCP workflow tools are read-only in V1** (`agent_workflow_validate`, `agent_workflow_status`, `agent_workflow_report`). `run` is CLI-only.

## Generating a workflow — example

User: "Use RagCode to orchestrate this: Gemini creates the plan, Claude analyzes and breaks down tasks, Codex implements the code, then Claude reviews code quality."

You should:

1. Generate the JSON spec (preserving the exact role assignment) and write it to `.ragcode/agents/workflows/gemini-claude-codex-review.json`.
2. Tell the user to run:
   ```bash
   ragcode agents validate . --workflow .ragcode/agents/workflows/gemini-claude-codex-review.json
   ragcode agents run . --workflow .ragcode/agents/workflows/gemini-claude-codex-review.json --task "<user task>" --dry-run
   ```
3. Only suggest the real `run` (without `--dry-run`) after validation passes and the user confirms.

## Cross-links

- **ragcode-context**: use `get_context` / `find_owner` / `find_reuse_candidates` / `related_tests` / `explain_impact` to enrich task envelopes. Workflow nodes receive RagCode context automatically when an engine is available.
- **ragcode-memory**: important workflow decisions can be persisted via `memory_write` after a run completes.
