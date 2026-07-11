# RagCode Agents Skill

[![skills.sh](https://skills.sh/b/MarshallEriksen-Neura/ragcode-skills)](https://skills.sh/MarshallEriksen-Neura/ragcode-skills)

Reusable agent skill for using [RagCode](https://github.com/MarshallEriksen-Neura/ragcode) as a multi-agent workflow orchestrator.

The `ragcode-agents` skill teaches coding agents to turn a natural-language collaboration pattern (e.g. "Gemini plans, Claude breaks down, Codex implements, Claude reviews") into a validated DAG workflow spec, then run it through the RagCode agent workflow engine with review gates, bounded fix loops, and an append-only run ledger.

This skill is published together with `ragcode-context` and `ragcode-memory` in the `MarshallEriksen-Neura/ragcode-skills` bundle. Default `npx skills add MarshallEriksen-Neura/ragcode-skills` installs all three skills; use `--skill ragcode-agents` only when you want this skill alone.

## Install

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills
```

Install only this skill explicitly:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --skill ragcode-agents
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

The agents skill also requires one or more agent CLIs to be installed and configured in `.ragcode/agents/agents.json` (built-ins: codex, claude, gemini, grok). Initialize the scaffold with `ragcode agents init .`.

## What The Skill Does

- Turns a user's natural-language collaboration pattern into a validated workflow JSON spec, preserving explicit role-to-provider assignments.
- Routes read-only stages (`plan`, `breakdown`, `review`, `verify`) with `mode: "read_only"` and implementation stages (`implement`, `fix`) with `mode: "write"` and a `writePolicy`.
- Adds review gates with `onFail` + `maxLoops` whenever a reviewer is named.
- Always validates before run and prefers `--dry-run` first.
- Confines write nodes to `writePolicy.allowedPaths` with one workspace write node at a time.
- Pairs with `ragcode-context` (enriches task envelopes) and `ragcode-memory` (persists workflow decisions after a run).

## Verify

List skills in this repository:

```bash
npx skills add MarshallEriksen-Neura/ragcode-skills --list
```

Generate a one-off prompt without installing:

```bash
npx skills use MarshallEriksen-Neura/ragcode-skills --skill ragcode-agents
```

## Files

```text
SKILL.md                      # agent routing instructions for multi-agent orchestration
agents/openai.yaml            # OpenAI/Codex metadata
references/workflow-schema.md  # workflow spec schema, validation rules, and presets
references/cli.md              # agents CLI reference and error recovery
```
