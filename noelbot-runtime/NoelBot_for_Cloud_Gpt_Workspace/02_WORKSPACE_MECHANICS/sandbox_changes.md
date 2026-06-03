# sandbox-changes

Priority: P1  
Status: filled protocol v0.1  
Purpose: Summarize live workfolder changes since the last known checkpoint, baseline, or current manifest without mutating source.

## Mechanic reproduced from source repo

This protocol translates `changes` from `nanstey/agentic-tools` into the ChatGPT sandbox environment. The reusable mechanic is: inspect the current working state, group changes by intent, classify risk/lane, and report what changed before committing/checkpointing or continuing.

## When to use

- Before creating a checkpoint when the user wants to know what changed.
- Before claiming a live workfolder update is complete.
- After a sequence of artifact writes when a concise delta is needed.
- When workspace drift, stale pointers, misplaced ZIPs, or unexpected files are suspected.
- Before deciding whether a checkpoint is meaningful enough to create.

## Inputs

- Bound sandbox workfolder target from `bind-target`.
- Optional baseline: last checkpoint verification, checksums file, manifest, or user-provided reference point.
- Workspace lane rules: source, derived, operational, temp, checkpoint pointer.
- Current checkpoint policy and cadence state.
- Any current goal lock or explicit user scope.

## Required target binding

Run `bind-target` first when the artifact, report, workfolder, baseline, or validation target is not already explicitly bound. Record the binding method and lifecycle role before acting.

## Process

1. Bind the workfolder and baseline. If no baseline exists, declare this as an initial inventory rather than a diff.
2. List added, modified, removed, and unexpected files using filesystem inspection and available checksum records.
3. Classify each changed path by lane: source, derived, operational, temp, checkpoint pointer, or unknown.
4. Flag forbidden or suspicious changes, especially ZIPs inside the live workfolder, source snapshot mutation, missing source files, stale current pointers, or orphaned temp evidence.
5. Group changes by purpose rather than only by filename: protocol fills, audits, checksums, derived insights, manifests, cleanup, recovery, checkpoint operations.
6. Determine whether the changes are checkpoint-worthy under cadence, milestone, risk, or user-request policy.
7. Return a concise change summary with action routing: continue, checkpoint, repair, triage, or abstain.

## Allowed actions

- Inspect filesystem paths, file metadata, and checksum manifests.
- Read enough content to identify file role and purpose.
- Compare against a prior checksum manifest, checkpoint verification, or current pointer.
- Delete or move misplaced ZIPs only when an active workspace rule already forbids ZIPs in the live workfolder and cleanup is part of the requested operation or preflight guardrail.
- Create a change summary or audit report when durable tracking is useful.

## Forbidden actions

- Do not mutate verified upstream source snapshots.
- Do not create checkpoint ZIPs inside the live workfolder or repo.
- Do not act on ambiguous targets.
- Do not promote hypotheses into rules without review.
- Do not mutate verified upstream source snapshots as part of change inspection.
- Do not treat absence of a baseline as proof of no changes.
- Do not silently ignore unexpected ZIPs, missing source files, or stale current pointers.
- Do not classify generated analysis as source truth.
- Do not perform high-effect repair actions; route them to restore/triage protocols.

## Evidence before

- Bound workfolder path.
- Baseline identifier or statement that no baseline exists.
- Checkpoint cadence state if relevant.
- Scope of inspection.

## Evidence after

- Counts of added, modified, removed, and unexpected files.
- Lane classification summary.
- Forbidden-pattern findings, if any.
- Checkpoint-worthiness decision and reason.
- Safe next action.

## Abstain conditions

```text
workfolder_not_bound
baseline_ambiguous
source_snapshot_missing
checksum_manifest_missing_when_required
unexpected_high_risk_change
permission_missing_for_cleanup
filesystem_inspection_failed
```

## Output schema

```json
{
  "change_status": "PASS | WARN | BLOCKED | INITIAL_INVENTORY",
  "workfolder": "absolute path",
  "baseline": "checkpoint/checksum/manifest/null",
  "change_counts": {
    "added": 0,
    "modified": 0,
    "removed": 0,
    "unexpected": 0
  },
  "lane_summary": {
    "source": 0,
    "derived": 0,
    "operational": 0,
    "temp": 0,
    "checkpoint_pointer": 0,
    "unknown": 0
  },
  "findings": [],
  "checkpoint_recommended": false,
  "safe_next_action": "continue | checkpoint | repair | triage | abstain"
}
```

## Output

A concise decision-useful result plus links/paths to any durable artifacts created. If the protocol only routes work, output the route and required gates rather than performing downstream actions.
