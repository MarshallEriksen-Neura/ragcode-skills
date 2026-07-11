# RagCode Agents CLI Reference (for agents)

All commands operate on a repo root (use `.` for the current directory). `--json` is available on most commands for machine-readable output.

## Setup

```bash
ragcode agents init .    # create .ragcode/agents/ scaffold (agents.json, workflows/)
ragcode agents list .    # list configured agents and their adapters/providers
```

## Validate and run

```bash
ragcode agents validate . --workflow <path>                          # validate a workflow spec without running
ragcode agents run . --workflow <path> --task "<task description>" --dry-run   # dry-run: plan and report without executing
ragcode agents run . --workflow <path> --task "<task description>"              # real run
```

`--dry-run` resolves agents, validates the DAG, and reports the execution plan without spawning CLIs or writing to the workspace. Always run `validate` or `--dry-run` before a real run.

## Run lifecycle

```bash
ragcode agents status . --run <runId>     # current node, gate state, overall run status
ragcode agents report . --run <runId>     # full run report (node outputs, gate verdicts, ledger)
ragcode agents resume . --run <runId> --workflow <path> --task "<task>"   # resume a paused/failed run
ragcode agents cancel . --run <runId>     # cancel a running run (process tree cancellation)
```

## Common flags

| Flag | Commands | Description |
|---|---|---|
| `--workflow <path>` | validate, run, resume | Path to the workflow spec JSON. |
| `--task "<text>"` | run, resume, resume | Natural-language task description passed to agents. |
| `--dry-run` | run | Plan-only; no execution. |
| `--run <runId>` | status, report, resume, cancel | Run identifier from a prior `run`. |
| `--json` | most | Machine-readable output. |

## Error recovery

- **Validation failure** → fix the spec (see `references/workflow-schema.md` for rules), re-run `validate`.
- **Missing CLI** → install the provider CLI (codex/claude/gemini/grok) or configure a new agent in `.ragcode/agents/agents.json`.
- **Gate failure after `maxLoops`** → run stops fail-closed. Inspect `ragcode agents report`, fix the root cause, then `resume`.
- **Cancelled/paused run** → `ragcode agents resume` with the same workflow and task to continue from the last checkpoint.
