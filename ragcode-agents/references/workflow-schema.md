# RagCode Workflow Spec Schema (for agents)

A workflow spec is **JSON data** validated by RagCode before execution. It declares the agents (role → provider mapping), the DAG of workflow nodes, and an optional final gate.

## Top-level fields

| Field | Required | Description |
|---|---|---|
| `version` | yes | Schema version. Currently `1`. |
| `name` | yes | Workflow name (kebab-case). |
| `description` | yes | Human-readable summary of the collaboration pattern. |
| `agents` | yes | Map of agent alias → `{ provider, adapter, role }`. |
| `workflow` | yes | Ordered list of DAG nodes (see below). |
| `finalGate` | no | Terminal verification/artifact gate applied after the last node. |

## Agent definition

```json
"<alias>": { "provider": "codex", "adapter": "cli", "role": "implementation" }
```

- `provider` — agent id from `.ragcode/agents/agents.json` (built-ins: `codex`, `claude`, `gemini`, `grok`).
- `adapter` — execution adapter. Currently `cli`.
- `role` — free-form role label (e.g. `planning`, `implementation`, `code_review`).

## Workflow node

```json
{
  "id": "implement",
  "agent": "executor",
  "kind": "implement",
  "mode": "write",
  "dependsOn": ["breakdown"],
  "input": ["tasks.json"],
  "output": ["patch.diff", "test-report.json"],
  "writePolicy": {
    "mode": "workspace",
    "allowedPaths": ["src/**", "tests/**", "docs/**"]
  },
  "gate": {
    "type": "review",
    "passRequired": true,
    "onFail": "implement_fix",
    "maxLoops": 3
  }
}
```

| Field | Required | Description |
|---|---|---|
| `id` | yes | Unique node identifier. |
| `agent` | yes | Alias from `agents` map. |
| `kind` | yes | `plan` \| `breakdown` \| `implement` \| `fix` \| `review` \| `verify` \| `custom`. |
| `mode` | yes | `read_only` \| `write`. Write-capable kinds (`implement`/`fix`/`custom`-with-write) require `write`. |
| `dependsOn` | no | Node ids this node waits on. Must exist; no cycles. |
| `input` | no | Artifacts from upstream nodes (file paths or output names). |
| `output` | no | Artifacts this node produces. |
| `writePolicy` | required for `mode: "write"` | `{ mode, allowedPaths }`. `allowedPaths` are repo-relative globs. |
| `gate` | no | Review/verification gate attached to this node. |

## Gate

```json
"gate": {
  "type": "review",
  "passRequired": true,
  "onFail": "implement_fix",
  "maxLoops": 3
}
```

| Field | Required | Description |
|---|---|---|
| `type` | yes | `review` \| `verification` \| `artifact`. |
| `passRequired` | no | If `true`, a failed gate blocks the run. |
| `onFail` | required when `passRequired: true` | Node id to jump to on failure. Must be a write-capable node. |
| `maxLoops` | required with `onFail` | Fix-loop iterations allowed (1..10). |
| `commands` | required for `verification` | Shell commands to run (e.g. `["npm run check", "npm test"]`). |
| `requiredArtifacts` | required for `artifact` | Artifact paths that must exist. |

## finalGate

Applied after the last workflow node completes:

```json
"finalGate": {
  "type": "verification",
  "commands": ["npm run check", "npm test"],
  "required": true
}
```

## Complete example

```json
{
  "version": 1,
  "name": "gemini-claude-codex-review",
  "description": "Gemini plans, Claude breaks down, Codex implements, Claude reviews.",
  "agents": {
    "planner":   { "provider": "gemini", "adapter": "cli", "role": "planning" },
    "analyst":   { "provider": "claude", "adapter": "cli", "role": "analysis_and_breakdown" },
    "executor":  { "provider": "codex",  "adapter": "cli", "role": "implementation" },
    "reviewer":  { "provider": "claude", "adapter": "cli", "role": "code_review" }
  },
  "workflow": [
    { "id": "plan",       "agent": "planner",  "kind": "plan",       "mode": "read_only", "output": "plan.md" },
    { "id": "breakdown",  "agent": "analyst",  "kind": "breakdown",  "mode": "read_only", "dependsOn": ["plan"], "input": ["plan.md"], "output": "tasks.json" },
    { "id": "implement",  "agent": "executor", "kind": "implement",  "mode": "write", "dependsOn": ["breakdown"], "input": ["tasks.json"],
      "output": ["patch.diff", "test-report.json"],
      "writePolicy": { "mode": "workspace", "allowedPaths": ["src/**", "tests/**", "docs/**"] } },
    { "id": "review",     "agent": "reviewer", "kind": "review",     "mode": "read_only", "dependsOn": ["implement"], "input": ["patch.diff", "test-report.json"],
      "output": "review.json",
      "gate": { "type": "review", "passRequired": true, "onFail": "implement_fix", "maxLoops": 3 } },
    { "id": "implement_fix", "agent": "executor", "kind": "fix",     "mode": "write", "dependsOn": ["review"], "input": ["review.json"],
      "output": ["patch.diff", "test-report.json"],
      "writePolicy": { "mode": "workspace", "allowedPaths": ["src/**", "tests/**", "docs/**"] } }
  ],
  "finalGate": { "type": "verification", "commands": ["npm run check", "npm test"], "required": true }
}
```

## Validation rules (RagCode enforces these — your spec must pass)

- All `agent` references in nodes must exist in `agents`.
- All `dependsOn` targets must exist; no cycles.
- `mode: "write"` nodes MUST declare `writePolicy.allowedPaths` (non-empty).
- `mode: "read_only"` nodes may NOT use write-capable kinds (`implement`/`fix`/`custom`-with-write).
- Gate `onFail` target MUST exist and be a write-capable node.
- A gate with `onFail` MUST declare `maxLoops` (1..10).
- `verification` gates MUST declare `commands`; `artifact` gates MUST declare `requiredArtifacts`.
- `provider` values must match an agent id in `.ragcode/agents/agents.json` (built-ins: codex, claude, gemini, grok).

## Presets

Use presets only when the user did NOT specify roles.

### `consult` — parallel read-only analysis, no writes

Multiple agents analyze in parallel; no implementation. Good for gathering perspectives.

### `plan-execute-review` — planner → executor → reviewer

Minimal 3-node workflow with one review gate.

### `gemini-claude-codex-review` — the realistic flow

The complete example above: Gemini plans, Claude breaks down, Codex implements, Claude reviews with a fix loop.
