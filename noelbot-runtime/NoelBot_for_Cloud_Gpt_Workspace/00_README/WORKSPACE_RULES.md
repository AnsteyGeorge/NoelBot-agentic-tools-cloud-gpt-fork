# Workspace Rules

Created: 20260603T004006Z UTC

## Folder roles

- `00_README/`: orientation, current goal, and workspace rules.
- `01_SOURCE_CAPTURE/`: source intake, source ledgers, and verification reports.
- `02_WORKSPACE_MECHANICS/`: sandbox diff, checkpoint, restore, and lineage protocols.
- `03_TARGET_BINDING/`: target-binding protocols for files, folders, repos, connector objects, and checkpoints.
- `04_VALIDATION_AND_REVIEW/`: validation, deviation triage, artifact review, duplication review, and decision-use review.
- `05_ADAPTATION_MECHANICS/`: mechanics models and adaptation notes.
- `06_AUTHORING/`: name checks, namespace registry, and workflow artifact scaffolding.
- `07_MANIFESTS/`: checksums, audits, and current pointers.
- `08_CHECKPOINTS_POINTER/`: pointer to external checkpoint root only. No ZIPs here.
- `protocols/`: protocol files for the first build set.

## Source / derived / operational separation

- Source evidence must be captured and verified before analysis.
- Derived analysis must not be treated as upstream source.
- Operational scripts/protocols must say what they may change and what they must not change.

## Checkpoint policy

Default checkpoint cadence: every 4th meaningful work turn.

Exceptions:
- explicit user request
- major milestone
- approaching chat/message limit
- risky or hard-to-reconstruct state change
- recovery event

Checkpoint ZIPs must live outside the active repo/live workfolder.
