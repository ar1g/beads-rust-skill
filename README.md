# beads-rust-skill

A skill that wraps the `br` CLI (beads_rust) to give AI agents
first-class issue-management capabilities directly inside a coding session.

## What it does

This skill teaches an agent how to create, triage, update, close, and
link issues using the `br` command-line tool. It covers the full lifecycle:
backlog grooming, priority management, dependency tracking, and auditable
closures -- all driven by structured, machine-readable output (`--json`).

## Credit

beads_rust is developed by **@Dicklesworthstone**:
<https://github.com/Dicklesworthstone/beads\_rust>

This skill is a thin orchestration layer on top of that tool. All issue
storage, querying, and mutation logic lives in beads_rust itself.

## Why this exists

Agents often need to manage project issues alongside code changes, but they
have no built-in awareness of any particular issue tracker. This skill bridges
that gap for repositories that use `.beads`-based tracking: it gives the agent
a reliable playbook for detecting `br`, resolving actor identity, reading state
before mutating, and reporting changes in a consistent format. The result is
safer, more predictable issue operations without requiring the user to dictate
every command.

## Install and usage

See [SKILL.md](SKILL.md) for the full skill definition, including:

- Prerequisites and `br` installation steps
- Core rules and actor resolution
- Standard workflows for triage and single-issue tasks
- Command reference and reporting format

## License

MIT
