# bind-target

Priority: P0  
Status: filled protocol v0.1  
Purpose: Bind the exact target file, folder, repo, connector object, checkpoint, generated artifact, or workflow object before ChatGPT acts.

## Mechanic reproduced from source repo

This protocol translates the `pr-info`, `changes`, and `check-skill-name` mechanics from `nanstey/agentic-tools` into the ChatGPT sandbox/connector/chat environment.

The reusable mechanic is:

```text
resolve the exact object of action before planning or executing the action
```

In the source repo, this prevents wrong-PR, wrong-branch, wrong-diff, or wrong-skill-name work. In this ChatGPT-native repo, it prevents wrong-folder, wrong-file, wrong-checkpoint, wrong-connector-object, wrong-artifact, or wrong-goal work.

## When to use

Use `bind-target` before any action that depends on a specific object, especially when the requested action can write, delete, restore, checkpoint, validate, summarize, adapt, or publish anything.

Use it even for read-only work when ambiguity would make the answer misleading.

## Target types

Supported target types:

```text
sandbox_workfolder
sandbox_file
sandbox_folder
generated_artifact
uploaded_file_reference
connector_object
github_repo
github_branch
github_pr
github_issue
checkpoint_zip
checkpoint_root
protocol_artifact
manifest_or_pointer
source_snapshot
derived_artifact
operational_artifact
```

## Inputs

- User request or current goal.
- Current live workfolder path, when applicable.
- Candidate target paths, names, URLs, connector references, or artifact links.
- Existing current pointers or manifests, when available.
- Source/derived/operational lane rules.
- Permission scope needed by the requested action.

## Binding process

1. **Extract target intent.** Identify the object the user is asking ChatGPT to read, write, validate, restore, adapt, checkpoint, or otherwise act on.
2. **List candidate targets.** Search or inspect only enough context to identify candidate objects.
3. **Classify target type.** Determine whether the candidate is source, derived, operational, checkpoint, connector, GitHub, generated, uploaded, or unknown.
4. **Check ambiguity.** If zero or multiple plausible targets exist, stop with a typed abstain state.
5. **Check authority and effect.** Identify whether the requested action is read-only, local write, checkpoint, restore, remote write, external effect, or destructive.
6. **Bind exact target.** Record the target identifier and binding method.
7. **Route next action.** Return the safe next action and gates required before execution.

## Binding record schema

A successful binding should be expressible as:

```json
{
  "binding_status": "BOUND",
  "target_type": "sandbox_file",
  "target_identifier": "relative/or/absolute/path/or/connector/id",
  "target_display_name": "human readable name",
  "binding_method": "user_provided | current_pointer | manifest_lookup | filesystem_inspection | connector_lookup | github_lookup | derived_from_context",
  "binding_confidence": "high | medium | low",
  "candidate_count": 1,
  "current_lifecycle_role": "source | derived | operational | checkpoint | temp | external_reference | unknown",
  "requested_effect_surface": "E0_read | E1_plan | E2_local_write | E3_local_state | E4_git_write | E5_remote_write | E6_history_rewrite | E7_destructive | E8_external_side_effect",
  "permission_scope_required": "none | read | local_write | checkpoint | restore | remote_write | destructive | external_effect",
  "required_next_gates": [],
  "safe_next_action": "read | summarize | validate | write_derived | ask_user | abstain | checkpoint | restore | execute_only_after_permission"
}
```

## Allowed actions

- Inspect filenames, manifests, current pointers, and relevant metadata.
- Read enough content to disambiguate target identity.
- Use connector/GitHub lookup tools when the target lives outside sandbox.
- Create a binding record in analysis output or a durable audit when the binding governs later work.
- Route to another protocol after binding succeeds.

## Forbidden actions

- Do not act on ambiguous targets.
- Do not mutate verified upstream source snapshots.
- Do not create checkpoint ZIPs inside the live workfolder or repo.
- Do not use a checkpoint ZIP as the live source of truth when an extracted live workfolder exists.
- Do not treat a generated analysis artifact as upstream source.
- Do not perform E4+ actions without explicit permission and target binding.
- Do not promote hypotheses into rules without review.

## Evidence before

Record at least:

- User request or current goal reference.
- Candidate target list or why only one target was possible.
- Binding method.
- Target type and lifecycle role.
- Required permission scope.
- Any ambiguity or excluded candidates.

## Evidence after

Record at least:

- Binding status.
- Exact target identifier.
- Safe next action.
- Required gates before action.
- Abstain code if binding failed.

## Abstain conditions

Return one of these instead of acting:

```text
target_not_found
multiple_targets
ambiguous_target
target_lifecycle_unknown
source_unverified
permission_missing
unsafe_effect_surface
current_pointer_missing
connector_lookup_failed
github_lookup_failed
validation_failed
```

## Output

The protocol outputs a concise binding result, not the final downstream action.

Example output:

```json
{
  "binding_status": "BOUND",
  "target_type": "sandbox_folder",
  "target_identifier": "/mnt/data/.../08_chatgpt_sandbox_agentic_tools_repo",
  "binding_method": "current_goal_and_filesystem_inspection",
  "requested_effect_surface": "E2_local_write",
  "safe_next_action": "write_derived_protocol_update",
  "required_next_gates": ["source_snapshot_read_only", "post_write_checksum"]
}
```
