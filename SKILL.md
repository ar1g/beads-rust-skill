---
name: beads-rust
description: Manage local-first issues with the beads_rust CLI (`br`). Use when creating or triaging issues, finding ready work, claiming or updating work, managing dependencies, diagnosing blocked queues, or synchronizing a repository's `.beads` SQLite/JSONL state.
license: MIT
metadata:
  author: local
  acknowledgements: Built on beads_rust by @Dicklesworthstone — https://github.com/Dicklesworthstone/beads_rust
  version: "1.1.0"
  domain: project-management
  triggers: br, beads, beads_rust, issue triage, backlog, dependencies, ready work
  role: specialist
  scope: operations
  output-format: commands
---

# Beads Rust

Use `br` for issue state in repositories that contain a `.beads` workspace. `br` is local-first: SQLite is the primary store and JSONL is the Git-friendly export.

## Prerequisite

Check before using the workflow:

```bash
command -v br >/dev/null 2>&1 && br version
```

If `br` is missing, tell the user and request approval before running the recommended installer:

```bash
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/beads_rust/main/install.sh?$(date +%s)" | bash
```

Do not infer permission to pipe a downloaded script into a shell. If the user prefers a manual installation, offer:

```bash
cargo install --git https://github.com/Dicklesworthstone/beads_rust.git beads_rust --locked
```

Building from source requires the repository's pinned Rust nightly toolchain. Verify the installed binary with `which br` and `br version`; multiple installs can leave an old binary earlier in `PATH`.

## Operating Rules

1. Use `br`, never the older `bd` command.
2. Discover the workspace with `br where --json` and inspect targeted issues before mutating them.
3. Prefer `--json` for agent parsing. Use `--format toon` when reduced token usage materially helps.
4. Resolve the mutation actor at runtime and pass `--actor`:

   ```bash
   ACTOR="${BR_ACTOR:-assistant}"
   ```

5. Pass explicit issue IDs. Some commands can fall back to the last-touched issue, which is unsafe in automation.
6. Keep mutations scoped to the request and read back the resulting state.
7. Successful mutations auto-flush `.beads/issues.jsonl` by default. `br sync --flush-only` is an idempotent final export check, and is necessary after `--no-auto-flush`, related configuration, or recovery work.
8. `br` does not commit, push, pull, or install Git hooks. Do not perform Git handoff unless the user requested it.
9. Most issue operations stay inside `.beads`, but explicit commands such as `br agents`, `br config edit/set`, `br completions -o`, `br doctor --repair`, and `br upgrade` can write elsewhere or update the binary. Treat those side effects according to the user's request.

For routine commands, prefix `RUST_LOG=error` if dependency logs would pollute structured output.

## Discover the Current Command Contract

The CLI is authoritative when installed behavior may differ from this skill:

```bash
br capabilities --format json
br capabilities --format json --command update
br robot-docs guide
br schema commands --format json
```

Use `br <command> --help` for flags not covered here. Do not guess a renamed flag or output shape.

## Verify the Workspace

Before a mutation batch, capture only the context needed for the task:

```bash
br where --json
br ready --json
br blocked --json
br list --status open --sort priority --json
```

Stop before mutating if workspace discovery is wrong or ambiguous. `br sync --status --json` checks DB/JSONL sync state without inspecting Git; use `br vcs-status --json` only when explicit Git visibility is relevant.

## Standard Workflow

For one issue:

1. Run `br show <id> --json`.
2. Apply the smallest requested mutation with `--actor "$ACTOR"` and `--json`.
3. Run `br show <id> --json` again.
4. Recheck `ready` or `blocked` when status or dependencies changed.
5. Report the exact mutation and verification result.

Claim work atomically:

```bash
ACTOR="${BR_ACTOR:-assistant}"
br update --actor "$ACTOR" <id> --claim --json
```

`--claim` assigns the actor and moves the issue to `in_progress`; it refuses blocked work unless `--force` is explicitly justified. Repositories may define custom statuses, transition comments, required acceptance criteria, gates, or capacity limits in `.beads/policy.yaml`. Follow the policy error evidence rather than bypassing it. When required, bind a fresh comment to the transition with `--transition-comment`.

Multi-target lifecycle commands are atomic within one repository. Routed operations spanning repositories use independent transactions, so do not report cross-repository atomicity.

## Triage

Classify each targeted issue once:

- `implemented`: close with commit, PR, path, or verified behavior as evidence.
- `out-of-scope`: close with a concrete boundary reason.
- `needs-clarification`: add a comment containing the unanswered question.
- `actionable`: keep open and correct only its status, priority, labels, parent, or dependencies.

For a large batch, inspect each issue, apply small batches, then recompute `ready`, `blocked`, and the open list. Do not modify unrelated issues or invent closure evidence.

## Create Issues

Bug descriptions should include reproduction, expected and actual behavior, environment, and log/crash pointers. Tasks and features should include objective, acceptance criteria, constraints, non-goals, and known dependencies.

```bash
ACTOR="${BR_ACTOR:-assistant}"
br create --actor "$ACTOR" "<title>" --type bug --priority 1 \
  --description "<repro, expected/actual, environment, evidence>" --json
```

Use `--description-file <path>` for multi-paragraph Markdown that would be fragile to shell-quote. Use `br q --actor "$ACTOR" "<title>"` only for intentionally minimal capture.

## Dependencies

`br dep add <issue> <depends-on>` means the first issue is blocked by the second:

```bash
br dep add <issue-id> <depends-on-id> --type blocks --json
br dep remove <issue-id> <depends-on-id> --json
br dep list <issue-id> --direction both --format json
br dep cycles --blocking-only --json
br blocked --json
```

Add `blocks` only for genuine execution ordering. Check both issues first, verify queue impact afterward, and keep the blocking graph cycle-free.

## Close, Reopen, Defer, and Delete

Use dedicated lifecycle commands rather than `br update --status closed`:

```bash
ACTOR="${BR_ACTOR:-assistant}"
br close --actor "$ACTOR" <id> --reason "<specific evidence>" --json
br reopen --actor "$ACTOR" <id> --reason "<why work resumed>" --json
br defer --actor "$ACTOR" <id> --until <date> --json
br undefer --actor "$ACTOR" <id> --json
```

If closure confidence is low, comment instead. `br delete` creates a tombstone and is not a substitute for closing completed or rejected work; use it only when deletion was specifically intended.

## Diagnose Hidden Work

If `br ready --json` is unexpectedly empty, inspect rather than reclaiming automatically:

```bash
br coordination status --json
br blocked --json
br dep cycles --blocking-only --json
```

`coordination status` is read-only. Stale claim evidence may justify a later user-scoped update, but it does not authorize automatic reclamation.

## Sync and Recovery

Never run bare `br sync`; choose an explicit direction or diagnostic mode. Ordinary mutations already auto-flush JSONL, so use:

```bash
br sync --status --json       # read-only DB/JSONL status
br sync --flush-only          # DB -> JSONL final export check
br sync --import-only         # JSONL -> DB after external JSONL changes
br sync --merge               # three-way merge divergent DB and JSONL state
```

Preview recovery operations and review their evidence before applying them. Do not use `--force`, `--force-db`, `--force-jsonl`, or `--rebuild` without establishing which state is authoritative and confirming that the resulting overwrite is within scope.

See [references/command-cookbook.md](references/command-cookbook.md) for conditional command patterns, recovery modes, and diagnostics.

## Report Results

Include:

- changed issue IDs and exact mutations;
- reasons and evidence for meaningful changes;
- verification performed and any `ready`/`blocked` impact;
- policy, coordination, sync, or user-input follow-ups still needed.

Do not claim that issue changes were committed or pushed unless those separate Git operations were requested and verified.
