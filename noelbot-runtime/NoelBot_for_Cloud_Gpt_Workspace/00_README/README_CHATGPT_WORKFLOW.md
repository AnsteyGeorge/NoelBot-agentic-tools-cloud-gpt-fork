# ChatGPT Sandbox Agentic Tools Repo

Version: usable v0.1  
State authority: `07_MANIFESTS/current_pointers/CURRENT_WORKSPACE_STATE.json`  
Created: 20260603T004006Z UTC  
Updated: 20260603T011945Z UTC

This repo is a ChatGPT-native adaptation of the verified `nanstey/agentic-tools` source snapshot and the W0-W4 adaptation analysis.

Current status: **usable v0.1**. The first build set is complete, dry-simulated, and has already been used for one low-risk real operation: creating `QUICKSTART.md`.

Its purpose is to reproduce the useful mechanics of the original repo inside a ChatGPT workflow environment:

- chat interface as coordinator
- sandbox filesystem as live work area
- connector tools as target/action surfaces
- generated artifacts as durable outputs
- external checkpoint ZIPs as commit-like recovery points
- user approvals as permission gates

## Current state and roots

The authoritative current-state object is:

```text
07_MANIFESTS/current_pointers/CURRENT_WORKSPACE_STATE.json
```

Root context:

```json
{
  "outer_live_workfolder_root": "/mnt/data/agentic_tools_inventory_20260602T083347Z",
  "nested_chatgpt_repo_root": "/mnt/data/agentic_tools_inventory_20260602T083347Z/08_chatgpt_sandbox_agentic_tools_repo",
  "source_snapshot_root": "/mnt/data/agentic_tools_inventory_20260602T083347Z/00_source_repo_snapshot/agentic-tools",
  "external_checkpoint_root": "/mnt/data/agentic_tools_inventory_checkpoints_20260602T083347Z"
}
```

Path rule: source-reference paths below are relative to the **outer live workfolder root**, not the nested ChatGPT repo root, unless explicitly marked otherwise.

## Non-negotiable rules

1. Do not mutate upstream verified source snapshots.
2. Do not place checkpoint ZIPs inside this repo or inside the live workfolder.
3. Bind targets before acting on files, folders, connectors, repos, checkpoints, or generated artifacts.
4. Treat validation findings as routing input, not automatic repair orders.
5. Use checkpoint-as-commit only when cadence, risk, user request, or milestone requires it.
6. Keep source, derived, operational, temp, and checkpoint artifacts separated.
7. Current-state claims should project from `CURRENT_WORKSPACE_STATE.json`, not compete with it.

## Source pattern library

The verified source repo lives at:

```text
00_source_repo_snapshot/agentic-tools/
```

It remains read-only. It is the source pattern library, not the implementation target.

## ChatGPT-native repo role

This repo lives at:

```text
08_chatgpt_sandbox_agentic_tools_repo/
```

It is the implementation target for ChatGPT-native workflow protocols derived from the source repo mechanics.

## First build set

The first build set includes:

```text
P0 bind-target
P0 source-capture-verify
P1 sandbox-changes
P1 contract-deviation-triage
P1 artifact-reviewer
P2 checkpoint-current-workfolder
P2 check-artifact-name
P3 make-workflow-artifact
```

These are tracked in:

```text
07_MANIFESTS/current_pointers/CURRENT_FIRST_BUILD_SET.json
```

## Quickstart

Start with:

```text
QUICKSTART.md
```

Then read:

```text
07_MANIFESTS/current_pointers/CURRENT_WORKSPACE_STATE.json
07_MANIFESTS/current_pointers/CURRENT_FIRST_BUILD_SET.json
```

## Checkpoint policy

Checkpoint ZIPs are external only:

```text
/mnt/data/agentic_tools_inventory_checkpoints_20260602T083347Z
```

This repo may contain only a pointer to the checkpoint root, not checkpoint ZIPs themselves.
