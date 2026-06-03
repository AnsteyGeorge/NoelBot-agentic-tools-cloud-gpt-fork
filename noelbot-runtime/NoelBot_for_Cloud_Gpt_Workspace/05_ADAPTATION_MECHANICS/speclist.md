# speclist

Priority: C1  
Status: filled protocol v0.1  
Purpose: Convert a report, audit, review, discovery note, or analysis into an execution-ready implementation checklist with explicit assumptions, risks, validation steps, and open questions.

## Mechanic reproduced from source repo

This protocol faithfully translates `skills/dev/speclist/SKILL.md` from `nanstey/agentic-tools` into the ChatGPT-orchestrated workspace.

The reusable mechanic is:

```text
turn free-form findings into an ordered, verifiable implementation spec without inventing facts or hiding blockers
```

In the source repo, `speclist` converts reports into implementation checklists for code work. In this ChatGPT-orchestrated repo, it converts chat analysis, artifact reviews, audit reports, connector findings, GitHub findings, and sandbox reports into bounded execution plans.

## When to use

Use `speclist` when the user has analysis or findings and wants a concrete implementation checklist.

Examples:

- Turn a self-review report into patch tasks.
- Turn a GitHub PR/CI/comment review into an implementation plan.
- Turn a workspace audit into a cleanup or repair checklist.
- Turn a design discussion into a scoped build plan.
- Turn an artifact-reviewer output into shippable work slices.

## Inputs

- Source report text, file, artifact, connector result, GitHub result, or chat section.
- Bound target from `bind-target` when the plan applies to a file, repo, workfolder, connector object, or GitHub object.
- Scope boundaries: in-scope systems/files/surfaces and out-of-scope items.
- Desired output depth: phase-level, task-level, file-level, or ticket-level.
- Constraints: timeline, risk tolerance, checkpoint cadence, permissions, external surfaces, and source/derived/operational lane rules.
- Definition of done: tests, validation, checkpoint, manifest/checksum update, review, approval, rollout, or backout.

## Required target binding

Run `bind-target` first when the source report, target repo/folder, GitHub PR, connector object, or affected artifact set is not already explicitly bound.

## Process

1. **Ingest the source report.** Read the report fully enough to preserve intent. State the core objective in one or two lines.
2. **Normalize findings.** Bucket raw material into goals, current state, gaps, proposed changes, risks, unknowns, and constraints.
3. **Split unrelated initiatives.** If the source mixes unrelated workstreams, stop and ask whether to split them or produce separate checklist sections.
4. **Derive requirements.** Convert findings into functional, operational, validation, documentation, and rollout requirements. Mark assumptions separately from confirmed facts.
5. **Sequence work.** Order tasks as pre-work, core implementation, integration/migration, validation, documentation, rollout/checkpoint, and handoff.
6. **Make items verifiable.** Each checklist item should have a completion condition and evidence requirement.
7. **Expose blockers.** Put unresolved blockers into Open Questions rather than burying them as tasks.
8. **Add risk controls.** Map each major risk to mitigation and detection signal.
9. **Validate coverage.** Confirm no important report finding was dropped. If an item is excluded, name it and why.
10. **Deliver the checklist.** Output a concise execution-ready plan, not a vague summary.

## Allowed actions

- Read source reports and bound target context.
- Reorganize findings into actionable implementation tasks.
- Create or update derived implementation plans and checklists.
- Label assumptions, blockers, out-of-scope items, and validation needs.
- Recommend next protocols to run.

## Forbidden actions

- Do not claim tasks were executed.
- Do not invent report facts.
- Do not silently upgrade assumptions into confirmed requirements.
- Do not hide blocking questions inside normal task items.
- Do not prescribe destructive, remote, or high-effect action without explicit permission and the relevant protocol gate.
- Do not mutate verified source snapshots.
- Do not create checkpoint ZIPs unless checkpoint cadence/risk/user request requires it.

## Evidence before

- Bound report/source target.
- Bound implementation target, if applicable.
- Scope and out-of-scope boundaries.
- Constraints and permission/effect surface.
- Definition of done or note that it must be inferred.

## Evidence after

- One-line objective.
- Ordered implementation checklist.
- Open Questions, blocking first.
- Risk Controls with mitigation and detection signal.
- Out-of-scope/deferred items.
- Validation/checkpoint requirements.

## Abstain conditions

```text
source_report_missing
source_report_too_vague
multiple_unrelated_initiatives_unsplit
target_scope_ambiguous
blocking_architecture_choice_unresolved
definition_of_done_missing_for_high_risk_work
permission_missing_for_high_effect_plan
```

## Output schema

```json
{
  "speclist_status": "PASS | PASS_WITH_OPEN_QUESTIONS | BLOCKED | ABSTAIN",
  "objective": "string",
  "scope": {"in_scope": [], "out_of_scope": []},
  "checklist_sections": [],
  "blocking_open_questions": [],
  "non_blocking_open_questions": [],
  "risk_controls": [],
  "validation_plan": [],
  "recommended_next_protocol": "string | null"
}
```

## Output

A markdown implementation checklist by default, with JSON summary when durable machine-readable planning is needed.
