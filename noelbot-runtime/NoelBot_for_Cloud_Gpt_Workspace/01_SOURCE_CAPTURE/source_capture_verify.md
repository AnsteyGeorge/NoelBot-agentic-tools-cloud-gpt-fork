# source-capture-verify

Priority: P0  
Status: filled protocol v0.1  
Purpose: Capture source artifacts and verify provenance/checksums before analysis, adaptation, or downstream artifact generation.

## Mechanic reproduced from source repo

This protocol translates the source-capture discipline we derived from `nanstey/agentic-tools` into the ChatGPT sandbox/connector/chat environment.

The reusable mechanic is:

```text
before using material as source truth, capture it, identify where it came from, verify it as strongly as possible, and record what remains incomplete
```

In the source repo analysis, exact GitHub files were only treated as complete after local materialization and Git blob SHA verification. In this ChatGPT-native repo, the same principle applies to uploaded files, connector fetches, GitHub files, generated exports, and copied source snapshots.

## When to use

Use `source-capture-verify` before:

- Summarizing or analyzing an external repo, uploaded file, connector object, or source bundle.
- Building derived records, schemas, roadmaps, or adaptation notes from source material.
- Treating any file as a source-of-truth input.
- Restoring a source snapshot from checkpoint.
- Claiming capture is complete.

## Source types

Supported source types:

```text
github_file
github_blob
github_repo_snapshot
uploaded_file
connector_file
connector_object
web_source
sandbox_existing_source_folder
checkpoint_restored_source_snapshot
generated_source_candidate
manual_pasted_source
```

## Inputs

- Bound source target from `bind-target`.
- Intended capture scope.
- Expected source count, if known.
- Available provenance such as URL, connector ID, Git blob SHA, checksum, file name, upload metadata, or checkpoint verification.
- Destination source lane.
- Current source/derived/operational separation rules.

## Capture process

1. **Bind the source target.** Do not capture until target identity and scope are bound.
2. **Declare capture scope.** State whether the scope is one file, many files, a repo subset, a full repo snapshot, an uploaded archive, or a connector object set.
3. **Materialize locally when needed.** If downstream analysis needs stable bytes, create a local source snapshot or file copy.
4. **Record provenance.** Include source type, original locator, fetched time, capture method, and any known upstream hash or revision ID.
5. **Verify identity.** Use the strongest available check:
   - Git blob SHA for GitHub files.
   - SHA-256 for local files/archives.
   - Checkpoint verification JSON for restored snapshots.
   - Connector/file metadata when hashes are unavailable.
6. **Record incompleteness.** If a source is only partially captured, label it incomplete and do not promote downstream analysis as complete.
7. **Separate source from derived output.** Never mix derived records or analysis into the source snapshot folder.
8. **Create capture report.** Include paths, counts, verification status, missing items, and caveats.

## Output

A capture verification report should include:

```json
{
  "capture_status": "PASS | PASS_WITH_CAVEATS | FAIL_INCOMPLETE | FAIL_VERIFICATION",
  "source_type": "...",
  "bound_target": "...",
  "capture_scope": "...",
  "materialized_paths": [],
  "verification_method": "...",
  "verified_count": 0,
  "missing_count": 0,
  "caveats": [],
  "downstream_allowed": true
}
```

## Abstain conditions

Stop and ask/route when:

- Target is ambiguous.
- Source cannot be fetched or materialized.
- Hash/checksum verification fails.
- The user asks to analyze source that is not actually captured.
- Source and derived paths are mixed.
- The capture report would claim completeness without evidence.

## Forbidden actions

- Do not fabricate missing source files.
- Do not silently skip missing source paths.
- Do not mutate upstream source snapshots to make validation pass.
- Do not treat generated analysis as source evidence.
- Do not overwrite source snapshots without a recovery/lineage plan.
