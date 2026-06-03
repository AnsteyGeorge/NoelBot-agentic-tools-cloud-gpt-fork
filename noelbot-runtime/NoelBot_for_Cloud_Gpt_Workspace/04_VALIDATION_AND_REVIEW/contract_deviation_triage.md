# contract-deviation-triage

Priority: P1  
Status: filled protocol v0.1  
Purpose: Route validation warnings/failures into accepted exceptions, semantic review, repair candidates, or quarantine without treating validator output as automatic repair permission.

## Mechanic reproduced from source repo

This protocol translates the repo-derived Contract Deviation Register pattern. The reusable mechanic is: strict validation is a signal for triage, not an instruction to edit source.

## When to use

- After any schema, contract, convention, frontmatter, checksum, or structure validation produces WARN/FAIL.
- Before repairing an artifact because a validator flagged it.
- Before adapting a source artifact that has known contract deviations.
- When format compliance and functional safety may diverge.

## Inputs

- Bound validation report, audit, or finding set.
- Bound artifact(s) referenced by the validation findings.
- Lifecycle role of each artifact: source, derived, operational, temp, checkpoint, or external reference.
- Validator rule or expected contract that triggered each finding.
- Available semantic records, deviation registers, and source evidence.

## Required target binding

Run `bind-target` first when the artifact, report, workfolder, baseline, or validation target is not already explicitly bound. Record the binding method and lifecycle role before acting.

## Process

1. Bind the validation report and affected artifact targets.
2. Separate finding type: format, function, authority, evidence, handoff, scope, runtime, or freshness.
3. Determine artifact lifecycle role. Source snapshots are evidence and normally do not get repaired in place.
4. Estimate risk if ignored and risk if repaired.
5. Classify likely interpretation: true_defect, intentional_exception, thin_wrapper, specialized_workflow, external_dependency, snapshot_limitation, stale_reference, or unknown.
6. Choose recommended action: accept_exception, document_exception, semantic_review, adapt_after_semantic_review, repair_derived_artifact, repair_upstream_only, quarantine, or abstain.
7. Set downstream routing: proceed, proceed_with_caveat, blocked_before_adaptation, blocked_before_execution.
8. Write a concise deviation register when findings will affect future decisions.

## Allowed actions

- Read validation reports and affected artifacts.
- Create or update derived deviation registers.
- Recommend repairs to derived/operational artifacts when evidence supports it.
- Mark source artifacts as do_not_modify while preserving adaptation caveats.
- Route downstream work to semantic review or adaptation after review.

## Forbidden actions

- Do not mutate verified upstream source snapshots.
- Do not create checkpoint ZIPs inside the live workfolder or repo.
- Do not act on ambiguous targets.
- Do not promote hypotheses into rules without review.
- Do not repair verified external source snapshots merely because validation fails.
- Do not treat every WARN as a blocker or every FAIL as a defect.
- Do not suppress findings without a rationale.
- Do not promote a deviation exception into a general rule without review.
- Do not execute high-effect workflows while their deviation status is unresolved.

## Evidence before

- Validation report path or finding source.
- Affected artifact paths or identifiers.
- Validator rule or expected contract.
- Lifecycle role and source verification status.
- Existing deviation records, if any.

## Evidence after

- Triage classification per finding.
- Recommended action per artifact/finding.
- Repair permission status.
- Downstream routing status.
- Register path if durable output was created.

## Abstain conditions

```text
validation_report_not_bound
affected_artifact_not_bound
source_status_unknown
finding_type_unknown
repair_scope_ambiguous
conflicting_authority
semantic_context_missing
```

## Output schema

```json
{
  "triage_status": "PASS_TRIAGED | WARN_UNRESOLVED | BLOCKED | ABSTAIN",
  "findings_total": 0,
  "classifications": {
    "accepted_exception": 0,
    "semantic_review": 0,
    "repair_candidate": 0,
    "quarantine": 0
  },
  "source_repairs_allowed": false,
  "downstream_allowed": [],
  "downstream_blocked": [],
  "safe_next_action": "proceed | semantic_review | repair_derived | quarantine | abstain"
}
```

## Output

A concise decision-useful result plus links/paths to any durable artifacts created. If the protocol only routes work, output the route and required gates rather than performing downstream actions.
