# check-artifact-name

Priority: P2  
Status: filled protocol v0.1  
Purpose: Check proposed ChatGPT workflow artifact names against the local namespace and reserved terms before creating new reusable protocols, registries, manifests, or helper artifacts.

## Mechanic reproduced from source repo

This protocol translates `.agents/skills/check-skill-name/SKILL.md` and `known-names.json` into the ChatGPT sandbox repo.

The core mechanic is namespace collision prevention: before making a new callable or reusable artifact, bind the proposed name, compare it against known names and local existing artifacts, and either approve, reject, or suggest alternatives.

## When to use

Use this before:

- creating a new protocol,
- adding a new workflow artifact,
- adding a new callable command-like name,
- renaming an existing artifact,
- adding a new registry entry,
- scaffolding via `make-workflow-artifact`.

## Inputs

- Proposed artifact name.
- Intended artifact purpose.
- Intended lane/domain folder.
- Artifact type: protocol, registry, manifest, audit, source-ledger, review, checklist, helper, or other.
- Current namespace registry.
- Current known artifact names.
- Existing filenames and protocol mirror names in the repo.

## Required target binding

Before deciding, bind:

```json
{
  "candidate_name": "lower-kebab-case candidate",
  "candidate_slug": "normalized slug",
  "target_domain": "folder or workflow family",
  "intended_effect": "what the artifact should help ChatGPT do",
  "registry_paths": [
    "06_AUTHORING/namespace_registry.json",
    "06_AUTHORING/known_artifact_names.json"
  ]
}
```

If the intended effect is unclear, stop and ask for clarification or propose a more descriptive name.

## Allowed actions

- Normalize the candidate into lower-kebab-case.
- Check exact name collisions against local registry names.
- Check near collisions against existing protocol names and filenames.
- Check reserved source references when available.
- Suggest safer alternatives.
- Return a verdict: `APPROVED`, `REJECTED_COLLISION`, `REVISE_FOR_CLARITY`, or `NEEDS_HUMAN_REVIEW`.
- Update the namespace registry only if the user is actively creating or reserving the artifact as part of an authorized workflow.

## Forbidden actions

- Do not create the artifact as part of this check unless explicitly combined with `make-workflow-artifact` after approval.
- Do not overwrite or rename existing artifacts.
- Do not mutate verified upstream source snapshots.
- Do not approve vague names like `helper`, `review`, `tool`, `thing`, or `new-protocol` without a purpose-specific qualifier.
- Do not approve names that imply high-effect authority unless the proposed artifact includes target-binding and permission gates.

## Evidence before

Collect:

- Candidate name and normalized slug.
- Purpose statement.
- Registry entries checked.
- Existing file/protocol names checked.
- Collision or similarity findings.

## Evidence after

Return:

```json
{
  "candidate_name": "string",
  "normalized_slug": "string",
  "verdict": "APPROVED | REJECTED_COLLISION | REVISE_FOR_CLARITY | NEEDS_HUMAN_REVIEW",
  "collisions": [],
  "near_collisions": [],
  "suggested_alternatives": [],
  "may_proceed_to_make_workflow_artifact": true
}
```

If the registry is updated, record the changed file path and checksum.

## Abstain conditions

- `candidate_missing`: no name was provided.
- `purpose_missing`: intended effect is unclear.
- `registry_missing`: namespace registry cannot be found.
- `ambiguous_target_domain`: cannot tell which artifact family the name belongs to.
- `collision_found`: exact collision exists.
- `high_effect_name_without_gates`: name implies risky action but no target-binding/permission design is provided.

## Naming rules

- Prefer lower-kebab-case for callable/protocol names.
- Use effect-oriented names, not vague nouns.
- Prefer `verb-object` or `domain-action` shapes, for example:
  - `bind-target`
  - `source-capture-verify`
  - `checkpoint-current-workfolder`
  - `contract-deviation-triage`
- Avoid vendor-specific names unless the artifact is intentionally vendor-specific.
- Avoid names that are too broad for the actual effect.

## Output

Return a concise name-check decision with approval status, reasons, collisions, and next safe action.

## Finished-quality test

This protocol succeeds if a future ChatGPT session can tell whether the proposed name is safe, why it was approved or rejected, and whether scaffolding may proceed.
