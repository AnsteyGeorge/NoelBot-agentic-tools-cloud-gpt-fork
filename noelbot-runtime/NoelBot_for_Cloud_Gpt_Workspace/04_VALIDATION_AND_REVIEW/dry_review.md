# dry-review

Priority: C1  
Status: filled protocol v0.1  
Purpose: Find true duplication, near-duplication, repeated decision logic, and consolidation opportunities while explicitly avoiding false coupling from incidental similarity.

## Mechanic reproduced from source repo

This protocol faithfully translates `skills/code-quality/dry/SKILL.md` from `nanstey/agentic-tools` into the ChatGPT-orchestrated workspace.

The reusable mechanic is:

```text
distinguish duplication worth consolidating from similarity that should stay separate
```

In the source repo, `dry` hunts for code duplication. In this repo, it can review code, protocols, generated artifacts, manifests, reports, matrices, state surfaces, and workflow definitions.

## When to use

- When similar files, reports, protocols, or code paths may be duplicative.
- Before creating another protocol that may overlap an existing one.
- When manifests, README sections, current-state objects, or pointers repeat state in conflicting ways.
- When a cleanup/consolidation opportunity might reduce future drift.
- When the user asks whether artifacts are duplicates or just related.

## Inputs

- Bound scan scope from `bind-target`.
- Scope type: current workfolder delta, named folder, protocol family, code directory, report set, or whole repo.
- Exclusion rules: generated files, checkpoint exports, vendored/dependency files, and intentionally retained evidence.
- Appetite: high-signal ranked pass by default, exhaustive only if requested.

## Required target binding

Run `bind-target` first unless the scope is explicit. For broad scans, confirm whether the target is a folder, protocol family, generated-report set, or whole repo.

## Process

1. **Confirm scan scope and exclusions.** State what will and will not be scanned.
2. **Gather candidate similarities.** Look for repeated names, repeated state fields, parallel structures, repeated decision rules, copy-paste text, and overlapping protocols.
3. **Classify each cluster.** Classify as true duplication, near-duplication, incidental similarity, superseded artifact, projection of current state, or intentionally retained evidence.
4. **Protect distinct lifecycle roles.** Do not merge source, derived, operational, temp, evidence, and checkpoint artifacts just because they look similar.
5. **Design consolidation.** For true duplication, propose helper, canonical state object, pointer, shared section, focused protocol, or retirement/supersession path.
6. **Rank by payoff.** Prioritize high drift risk, high repetition count, and low consolidation cost.
7. **Name deliberate non-findings.** Explain important similarities left separate and why.

## Allowed actions

- Read files and metadata in scope.
- Compare text, structure, purpose, lifecycle role, and current pointers.
- Produce ranked duplication/consolidation findings.
- Recommend merge, supersede, split, or leave-separate actions.

## Forbidden actions

- Do not edit, delete, merge, move, or checkpoint as part of the review.
- Do not merge artifacts with different lifecycle roles without explicit review.
- Do not propose catch-all junk-drawer files.
- Do not treat all repetition as bad; projections from one state authority may be intentional.
- Do not remove retained evidence merely because a newer summary exists.

## Evidence before

- Bound scan scope.
- Exclusion rules.
- Lifecycle roles in scope.
- Known current-state authority and manifest/pointer files.

## Evidence after

- Scope scanned and exclusions.
- Ranked true/near duplication findings.
- Deliberate non-findings.
- Proposed consolidation shape.
- Highest-payoff first action.

## Abstain conditions

```text
scan_scope_ambiguous
lifecycle_roles_unknown
source_vs_derived_boundary_unclear
exclusion_rules_missing_for_broad_scan
too_broad_without_sampling_permission
```

## Output schema

```json
{
  "dry_review_status": "PASS | PASS_WITH_FINDINGS | BLOCKED | ABSTAIN",
  "scope_scanned": [],
  "excluded": [],
  "ranked_findings": [],
  "deliberate_non_findings": [],
  "highest_payoff_action": "string | null",
  "recommended_next_protocol": "speclist | artifact-reviewer | make-workflow-artifact | null"
}
```

## Output

A ranked duplication/consolidation report. Include non-findings where false coupling would be harmful.
