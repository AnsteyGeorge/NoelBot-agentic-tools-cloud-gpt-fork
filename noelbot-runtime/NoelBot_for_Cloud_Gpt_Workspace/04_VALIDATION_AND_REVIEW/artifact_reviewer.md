# artifact-reviewer

Priority: P1  
Status: filled protocol v0.1  
Purpose: Review artifacts for purpose fit, decision value, duplication, provenance, and finished-quality impact before promoting or extending them.

## Mechanic reproduced from source repo

This protocol translates `code-reviewer`, `deep-review`, and `dry` into ChatGPT artifact-workspace mechanics. The reusable mechanic is: review for usefulness, risk, duplication, provenance, and decision value rather than surface completeness alone.

## When to use

- Before promoting an insight into a rule, gate, protocol, or source-of-truth pointer.
- Before creating another similar artifact when duplication may exist.
- Before adapting a workflow artifact into a target environment.
- When a report or protocol may be precision theater rather than decision-useful.
- After a batch of durable artifacts to confirm they improve finished quality.

## Inputs

- Bound artifact or artifact set from `bind-target`.
- Current goal and intended user/workflow need.
- Source evidence references and derived evidence references.
- Existing related artifacts, current pointers, and manifests.
- Finished-quality criteria and promotion status.

## Required target binding

Run `bind-target` first when the artifact, report, workfolder, baseline, or validation target is not already explicitly bound. Record the binding method and lifecycle role before acting.

## Process

1. Bind the artifact and classify its lifecycle role.
2. State the artifact purpose and decision it is supposed to improve.
3. Check provenance: source evidence, derived evidence, and whether claims overreach evidence.
4. Check decision value: what future action becomes easier, faster, or safer because this exists?
5. Check duplication: exact duplicate, overlapping but distinct, superseded, or complementary.
6. Check finished-quality dimensions: correctness, completeness, usability, cleanliness, recoverability, and decision value.
7. Check promotion risk: observation, finding, hypothesis, candidate rule, adopted rule, enforced gate.
8. Recommend action: keep, refine, merge, supersede, split, quarantine, promote_candidate, or retire.

## Allowed actions

- Read artifacts and nearby related records.
- Compare artifact purposes, claims, evidence refs, and current pointers.
- Create review reports or recommendations.
- Recommend merge/supersede/split actions for derived artifacts.
- Flag precision theater and unnecessary friction.

## Forbidden actions

- Do not mutate verified upstream source snapshots.
- Do not create checkpoint ZIPs inside the live workfolder or repo.
- Do not act on ambiguous targets.
- Do not promote hypotheses into rules without review.
- Do not mutate source snapshots during review.
- Do not delete or overwrite artifacts without explicit cleanup authorization.
- Do not equate length/detail with quality.
- Do not promote a hypothesis into a rule without a promotion gate.
- Do not call duplicated-but-distinct artifacts redundant without checking lifecycle context.
- Do not checkpoint only because a review occurred unless durable state changed and cadence/risk/request requires it.

## Evidence before

- Bound artifact path(s).
- Intended purpose and current goal.
- Related current pointers or manifests.
- Evidence refs used by the artifact.
- Known superseding or predecessor artifacts, if any.

## Evidence after

- Review verdict.
- Decision-use assessment.
- Duplication/supersession assessment.
- Promotion status recommendation.
- Action recommendation and rationale.

## Abstain conditions

```text
artifact_not_bound
purpose_unknown
evidence_refs_missing
related_artifacts_unavailable
promotion_authority_missing
duplication_uncertain
review_scope_too_broad
```

## Output schema

```json
{
  "review_status": "PASS | PASS_WITH_CAVEATS | NEEDS_REFINEMENT | DUPLICATIVE | BLOCKED | ABSTAIN",
  "decision_value": "high | medium | low | unclear",
  "duplication_status": "none | exact_duplicate | overlapping_distinct | superseded | uncertain",
  "promotion_recommendation": "do_not_promote | keep_candidate | promote_candidate | adopted_rule_review_needed",
  "recommended_action": "keep | refine | merge | split | supersede | quarantine | retire | abstain"
}
```

## Output

A concise decision-useful result plus links/paths to any durable artifacts created. If the protocol only routes work, output the route and required gates rather than performing downstream actions.
