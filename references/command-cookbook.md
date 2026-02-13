# Command Cookbook

Resolve actor at runtime for mutating operations (agent-executed):

```bash
ACTOR="${BR_ACTOR:-assistant}"
```

## Quick Inspection

```bash
br where
br ready --json
br blocked --json
br list --status open --sort priority --json
br show <id> --json
br search "<query>" --json
```

## Create

Bug:
```bash
ACTOR="${BR_ACTOR:-assistant}"
br create --actor "$ACTOR" --type bug --priority 1 --labels reliability \
  "<title>" --description "<summary + repro + expected/actual + env>" --json
```

Task:
```bash
ACTOR="${BR_ACTOR:-assistant}"
br create --actor "$ACTOR" --type task --priority 2 --labels ui,ux \
  "<title>" --description "<objective + acceptance criteria>" --json
```

Quick capture (when you just need an ID):
```bash
ACTOR="${BR_ACTOR:-assistant}"
br q --actor "$ACTOR" "<title>"
```

## Update

```bash
ACTOR="${BR_ACTOR:-assistant}"
br update --actor "$ACTOR" <id> --priority 2 --status in_progress --json
br update --actor "$ACTOR" <id> --add-label reliability --json
br update --actor "$ACTOR" <id> --parent <parent-id> --json
br update --actor "$ACTOR" <id> --claim --json
```

Bulk update (batch triage):
```bash
ACTOR="${BR_ACTOR:-assistant}"
br update --actor "$ACTOR" <id1> <id2> <id3> --priority 2 --add-label triage-reviewed --json
```

## Comments

```bash
ACTOR="${BR_ACTOR:-assistant}"
br comments add --actor "$ACTOR" <id> --message "<triage note / evidence>" --json
```

## Close / Reopen

```bash
ACTOR="${BR_ACTOR:-assistant}"
br close --actor "$ACTOR" <id> --reason "<specific reason with evidence>" --json
br close --actor "$ACTOR" <id> --reason "<reason>" --suggest-next --json
br close --actor "$ACTOR" <id> --reason "<reason>" --force --json
br reopen --actor "$ACTOR" <id> --reason "<reason for reopening>" --json
```

## Dependency Operations

```bash
br dep add <issue-id> <depends-on-id> --type blocks
br dep rm <issue-id> <depends-on-id>
br dep list <issue-id> --json
br dep tree <issue-id> --json
br dep cycles --json
```

## End-of-Triage Check

```bash
br ready --json
br blocked --json
br list --status open --sort priority --json
```

## Output Formats

`--json` is the default for machine-readable output. For reduced token usage, use `--format toon`:
```bash
br list --status open --sort priority --format toon
br show <id> --format toon
```

## Diagnostics

```bash
br stats --json
br stale --json
br lint --json
br sync --status
```

## Sync

```bash
br sync --flush-only
```
