# Command Cookbook

Use these patterns only when the corresponding operation is requested. Prefer `br capabilities --format json --command "<command>"` or `br <command> --help` when the installed version differs.

## Runtime Setup

```bash
ACTOR="${BR_ACTOR:-assistant}"
```

Prefix routine commands with `RUST_LOG=error` if Rust dependency logs interfere with machine-readable output.

## Inspect and Select Work

```bash
br where --json
br ready --json
br blocked --json
br list --status open --sort priority --json
br list --priority 0-1 --json
br show <id> --json
br search "<query>" --json
br stale --days 30 --json
br count --by status --json
br scheduler --limit 5 --candidate-limit 100 --format json
```

When ready work appears unexpectedly absent:

```bash
br coordination status --json
br dep cycles --blocking-only --json
```

Coordination diagnostics are read-only; do not auto-reclaim a claim solely because it looks stale.

## Create

```bash
ACTOR="${BR_ACTOR:-assistant}"
br create --actor "$ACTOR" "<title>" --type bug --priority 1 \
  --labels reliability \
  --description "<repro, expected/actual, environment, evidence>" --json

br create --actor "$ACTOR" "<title>" --type task --priority 2 \
  --description "<objective, acceptance criteria, constraints>" --json

br create --actor "$ACTOR" --title "<title>" \
  --description-file <path> --json

br q --actor "$ACTOR" "<title>"
```

Priorities are numeric: 0 critical, 1 high, 2 medium/default, 3 low, 4 backlog.

## Update and Claim

```bash
ACTOR="${BR_ACTOR:-assistant}"
br update --actor "$ACTOR" <id> --claim --json
br update --actor "$ACTOR" <id> --priority 2 --json
br update --actor "$ACTOR" <id> --status in_progress --json
br update --actor "$ACTOR" <id> --add-label reliability --json
br update --actor "$ACTOR" <id> --remove-label stale --json
br update --actor "$ACTOR" <id> --parent <parent-id> --json
br update --actor "$ACTOR" <id> --parent "" --json
```

Batch updates accept multiple IDs and are atomic within one repository:

```bash
br update --actor "$ACTOR" <id1> <id2> <id3> \
  --priority 2 --add-label triage-reviewed --json
```

If `.beads/policy.yaml` requires transition evidence, include it in the same mutation:

```bash
br update --actor "$ACTOR" <id> --status in_review \
  --transition-comment "<fresh evidence for this transition>" --json
```

Do not combine `--claim` with a redundant status/assignee update. Do not use `--force` to bypass a blocker or policy failure without explicit justification.

## Comments

```bash
ACTOR="${BR_ACTOR:-assistant}"
br comments add --actor "$ACTOR" <id> --message "<note or evidence>" --json
br comments add --actor "$ACTOR" <id> --file <path> --json
br comments list <id> --json
```

## Close, Reopen, Defer, and Delete

```bash
ACTOR="${BR_ACTOR:-assistant}"
br close --actor "$ACTOR" <id> --reason "<specific evidence>" --json
br close --actor "$ACTOR" <id1> <id2> --reason "<shared evidence>" --json
br reopen --actor "$ACTOR" <id> --reason "<reason>" --json
br defer --actor "$ACTOR" <id> --until tomorrow --json
br undefer --actor "$ACTOR" <id> --json
br delete --actor "$ACTOR" <id> --reason "<why a tombstone is intended>" --json
```

Use `close`, not `update --status closed`; use `delete` only when a tombstone is intended. Multi-target lifecycle mutations preflight and commit atomically within one repository.

## Dependencies and Graphs

```bash
br dep add <issue-id> <depends-on-id> --type blocks --json
br dep remove <issue-id> <depends-on-id> --json
br dep list <issue-id> --direction both --format json
br dep tree <issue-id> --json
br dep cycles --blocking-only --json
br graph <issue-id> --json                 # what this issue unblocks
br graph <issue-id> --dependencies --json  # what blocks this issue
```

For bulk imports, inspect the input first, then use `br dep import <path> --robot`. Verify `br dep cycles --blocking-only --json` and `br blocked --json` afterward.

## Labels

```bash
br label add <id> --label backend --json
br label add <id> --label auth --json
br label remove <id> --label urgent --json
br label list <id> --json
br label list-all --json
```

## Sync Status and Normal Handoff

Successful issue mutations auto-flush JSONL by default. These modes remain explicit:

```bash
br sync --status --json        # read-only DB/JSONL comparison; no Git probe
br sync --witness --robot      # deterministic, read-only JSONL witness
br sync --flush-only           # DB -> JSONL
br sync --import-only          # JSONL -> DB
br sync --merge                # base + DB + JSONL three-way merge
br vcs-status --json           # explicitly inspect JSONL Git visibility
```

Bare `br sync` is rejected. `br` never commits, pulls, or pushes. Perform Git handoff only when the user asks for it, and verify each affected repository separately when routed operations crossed workspaces.

## Recovery

For JSONL rows missing or newer in SQLite, start with the previewable additive path:

```bash
br sync --reconcile --dry-run
br sync --reconcile
```

For receipt-bound exact-ID additive recovery, review the plan and bind apply to its hash:

```bash
plan="$(br sync --reconcile-additive --robot)"
plan_sha256="$(printf '%s\n' "$plan" | jq -r .plan_sha256)"
br sync --reconcile-additive --apply \
  --expect-plan-sha256 "$plan_sha256" --robot
```

For divergent DB and JSONL edits:

```bash
br sync --merge
```

If semantic conflicts remain, establish which side is authoritative before choosing `--force-db`, `--force-jsonl`, or timestamp-based `--force`. `br sync --import-only --rebuild` removes database entries absent from JSONL and therefore requires explicit authority that JSONL should replace SQLite state.

## Diagnostics and Self-Description

```bash
br capabilities --format json
br capabilities --format json --command "update"
br robot-docs guide
br schema commands --format json
br doctor --json
br stats --json
br lint --status all --json
br config list
br where --json
br version --json
```

`br doctor --repair`, `br config edit/set`, `br agents`, `br completions -o`, and `br upgrade` mutate configuration, instruction files, completions, or the installed binary; do not treat them as diagnostics.

## Cross-Workspace Routing

Repositories can route issue prefixes through `.beads/routes.jsonl`. Route-aware commands resolve the target workspace and mutate its storage. A multi-repository request is not a distributed transaction: each target repository commits independently and needs separate verification and VCS handoff.
