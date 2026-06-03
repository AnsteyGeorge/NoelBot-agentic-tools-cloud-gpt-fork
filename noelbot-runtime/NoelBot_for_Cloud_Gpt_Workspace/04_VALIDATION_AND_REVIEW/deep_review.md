# deep-review

Priority: C1  
Status: filled protocol v0.1  
Purpose: Run a strict maintainability and structural-quality review focused on abstraction quality, simplification opportunities, sprawl, branching complexity, and design cleanliness.

## Mechanic reproduced from source repo

This protocol faithfully translates `skills/code-quality/deep-review/SKILL.md` from `nanstey/agentic-tools` into the ChatGPT-orchestrated workspace.

The reusable mechanic is:

```text
look for structural simplification and code/workflow judo, not just local cleanup
```

In the source repo, `deep-review` is an ambitious maintainability audit for code changes. In this repo, it applies to code, protocols, artifact systems, workspace governance, generated artifact structures, and ChatGPT-orchestrated workflow design.

## When to use

- When a change works but may make the system more complex.
- Before adopting a broad abstraction, protocol family, or workflow layer.
- When files, protocols, matrices, or reports grow large or repetitive.
- When branching rules, exception paths, or special cases are spreading.
- When the user asks for a harsh maintainability or design-quality review.

## Inputs

- Bound review target from `bind-target`.
- Scope: code diff, protocol family, artifact folder, workflow model, or generated report set.
- Current goal and finished-quality definition.
- Existing canonical helpers, protocols, layers, and state authority.

## Required target binding

Run `bind-target` first unless the review target is already explicit. If the review spans multiple independent systems, split the review or ask for scope choice.

## Process

1. **State the target and quality bar.** Identify what would count as a structural improvement.
2. **Look for simplification opportunities.** Ask whether a different model would delete branches, files, layers, flags, or repeated decisions.
3. **Check sprawl.** Identify files/artifacts/protocols that have grown beyond their role or have become junk drawers.
4. **Check spaghetti growth.** Flag scattered special cases, one-off branches, and mode flags added into unrelated flows.
5. **Check boundaries.** Look for logic in the wrong layer, weak type/contract boundaries, unclear ownership, and source/derived/operational leaks.
6. **Check abstraction quality.** Challenge wrappers, indirection, or generic machinery that does not earn its complexity.
7. **Prefer high-conviction findings.** Do not flood the review with nits when structural issues exist.
8. **Recommend remedies.** Prefer deletion, simplification, explicit state models, clearer ownership, or focused extraction over cosmetic cleanup.

## Allowed actions

- Read bounded targets and related context.
- Produce maintainability findings and structural recommendations.
- Recommend extraction, simplification, consolidation, or state-authority changes.
- Recommend follow-up with `speclist` when implementation planning is needed.

## Forbidden actions

- Do not edit, create, move, delete, checkpoint, stage, commit, push, or execute as part of review.
- Do not approve merely because behavior appears correct.
- Do not treat cosmetic nits as equivalent to structural findings.
- Do not invent architectural requirements not supported by the target context.
- Do not mutate source snapshots.

## Evidence before

- Bound review target and scope.
- Current goal or target quality bar.
- Relevant canonical layers/protocols/state objects.
- Known constraints and out-of-scope areas.

## Evidence after

- Structural verdict.
- Ranked maintainability findings.
- Simplification opportunities.
- Boundary/sprawl/abstraction concerns.
- Recommended implementation plan or next protocol.

## Abstain conditions

```text
review_scope_ambiguous
target_context_insufficient
multiple_unrelated_systems_unsplit
canonical_layer_unknown
quality_bar_unknown
```

## Output schema

```json
{
  "deep_review_status": "PASS | PASS_WITH_FINDINGS | NEEDS_RESTRUCTURE | BLOCKED | ABSTAIN",
  "structural_regression_risk": "low | medium | high | unclear",
  "top_findings": [],
  "simplification_opportunities": [],
  "boundary_issues": [],
  "sprawl_or_branching_concerns": [],
  "recommended_next_protocol": "speclist | artifact-reviewer | contract-deviation-triage | null"
}
```

## Output

A direct, ranked maintainability review. Focus on the biggest structural wins first.
