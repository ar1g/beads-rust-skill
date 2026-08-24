# beads-rust-skill

A skill that teaches AI agents to use the current `br` CLI (beads_rust) for
local-first issue management directly inside a coding session.

## What it does

This skill covers issue creation and triage, atomic claims and lifecycle
updates, dependency-aware ready/blocked queues, coordination diagnostics,
policy-aware transitions, explicit DB/JSONL sync and recovery, and auditable
closures. It defaults to structured `--json` output and uses the CLI's
self-describing capabilities when installed behavior has moved ahead.

## Credit

beads_rust is developed by **@Dicklesworthstone**:
<https://github.com/Dicklesworthstone/beads_rust>

This skill is a thin orchestration layer on top of that tool. All issue
storage, querying, and mutation logic lives in beads_rust itself.

## Why this exists

This skill gives the agent a reliable playbook for the Rust port of Steve
Yegge's classic beads issue tracker while preserving `br`'s non-invasive,
local-first model.

## Install and usage

See [SKILL.md](SKILL.md) for the full skill definition, including:

- Prerequisites and `br` installation steps
- Workspace discovery, actor resolution, and current command contracts
- Standard workflows for triage, claims, dependencies, and lifecycle changes
- JSONL auto-flush semantics, explicit sync/recovery modes, and reporting

## License

MIT
