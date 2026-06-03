# checkpoint-current-workfolder

Priority: P2  
Status: filled protocol v0.1  
Purpose: Create an external verified checkpoint ZIP for the active live workfolder according to cadence, risk, or explicit user request.

## Mechanic reproduced from source repo

This protocol translates the source repo's `commit` and change-preservation mechanics into the ChatGPT sandbox environment.

In the source repo, a commit records an intentional repo state. In this ChatGPT sandbox repo, a checkpoint records an intentional live workfolder state. The checkpoint is not the source of truth while work continues; it is a recovery/export snapshot of the live workfolder.

## When to use

Use this protocol when one of these conditions is true:

1. The user explicitly asks for a checkpoint or downloadable ZIP.
2. The configured cadence is due.
3. A risky or hard-to-reconstruct state change occurred.
4. Chat/message limits may interrupt continuity.
5. The live workfolder has reached a meaningful durable milestone.

Do not use this protocol after every small thought, note, or reversible analysis step.

## Inputs

- Bound live workfolder path.
- Bound external checkpoint root path.
- Current goal or milestone being checkpointed.
- Checkpoint reason: `user_request`, `cadence_due`, `risk_exception`, `message_limit_risk`, or `milestone`.
- Current manifest/checksum state when available.

## Required target binding

Before creating a checkpoint, bind:

```json
{
  "live_workfolder": "absolute path to active source-of-truth workfolder",
  "checkpoint_root": "absolute path outside the live workfolder",
  "checkpoint_filename": "UTC timestamped ZIP filename",
  "checkpoint_reason": "user_request | cadence_due | risk_exception | message_limit_risk | milestone"
}
```

The checkpoint root must be outside the live workfolder. If the target is ambiguous, stop.

## Allowed actions

- Inspect the live workfolder for ZIP recursion risk.
- Remove or report misplaced checkpoint ZIPs inside the live workfolder according to workspace rules.
- Create one checkpoint ZIP in the external checkpoint root.
- Verify the ZIP can be opened and contains the expected live workfolder contents.
- Compute SHA-256 for the checkpoint ZIP.
- Write a checkpoint verification JSON next to the ZIP.
- Update the checkpoint pointer or manifest when appropriate.
- Report the checkpoint link and verification result.

## Forbidden actions

- Do not create checkpoint ZIPs inside the live workfolder.
- Do not include previous checkpoint ZIPs inside the new checkpoint.
- Do not mutate verified upstream source snapshots.
- Do not silently overwrite a checkpoint without a timestamped filename unless explicitly acting on a pointer file.
- Do not create a checkpoint if the live workfolder target is ambiguous.
- Do not treat checkpoint ZIPs as the live source of truth while the live workfolder exists.

## Evidence before

Collect or report:

- Bound live workfolder path.
- Bound checkpoint root path.
- Checkpoint reason.
- Live workfolder ZIP count before checkpoint.
- Unexpected hidden-file check, if relevant.
- Existing checkpoint family state, if cleaning or replacing old checkpoint entries.

## Evidence after

Produce:

```json
{
  "checkpoint_zip": "path",
  "checkpoint_sha256": "hex sha256",
  "checkpoint_size_bytes": 0,
  "zip_open_test": "PASS | FAIL",
  "nested_zip_count": 0,
  "live_workfolder_zip_count_after": 0,
  "created_at_utc": "YYYYMMDDTHHMMSSZ"
}
```

## Abstain conditions

- `target_not_bound`: live workfolder or checkpoint root is unknown.
- `checkpoint_root_inside_live_workfolder`: would create recursion.
- `live_workfolder_missing`: the source-of-truth workfolder does not exist.
- `zip_open_test_failed`: produced ZIP cannot be verified.
- `unexpected_source_loss`: source files are missing and must be restored before checkpointing.
- `permission_missing`: user has not authorized checkpointing and no cadence/risk rule applies.

## Output

Return a concise checkpoint result with:

- checkpoint ZIP link/path,
- verification JSON link/path,
- reason for checkpoint,
- SHA-256,
- nested ZIP count,
- whether the live workfolder remained ZIP-free.

## Finished-quality test

This protocol succeeds only if a future ChatGPT session can restore or inspect the checkpoint without guessing what it represents, and if the live workfolder remains the active source of truth.
