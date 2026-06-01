---
name: speclist
description: Transform a free-form report into a concrete implementation-spec checklist by extracting scope, assumptions, constraints, tasks, risks, and validation steps, then rewriting them into an ordered, execution-ready plan with explicit open questions. Use when the user has analysis or findings and wants an actionable implementation checklist.
user-invocable: true
disable-model-invocation: false
---

# Speclist

## Core Contract

Use this skill when the user has a report (audit notes, bug analysis, review findings, postmortem, or discovery write-up) and wants an implementation-spec checklist they can execute.

Default scope: convert one report into one implementation checklist for a single codebase/workstream, preserving intent while making actions concrete and verifiable.

Treat `CLAUDE.md` / `AGENTS.md` in the target repository as the source of truth. If they conflict with this skill, follow them.

## Required Inputs

Gather or infer:

1. The source report text (or file/PR/issue link containing it).
2. Target scope boundaries (in/out of scope systems, files, or teams).
3. Expected output depth (high-level phases vs file-level task list).
4. Constraints (timeline, release window, risk tolerance, tooling/process rules).
5. Definition of done (tests, rollout, docs, approvals, monitoring, backout).

If the report is missing or too vague to extract actions, stop and ask for a clearer source before proceeding.

## Workflow

1. **Ingest and normalize**
   - Read the report fully and restate its core objective in 1-2 lines.
   - Extract raw items into buckets: goals, current state, gaps, proposed changes, risks, unknowns.
   - Stop-and-ask gate: if multiple unrelated initiatives are mixed together, ask whether to split into separate specs.

2. **Derive implementation requirements**
   - Convert findings into concrete requirements: functional, non-functional, migration, operational, and validation requirements.
   - Tag each requirement with rationale from the report (why this exists).
   - Mark assumptions vs confirmed facts; do not silently upgrade assumptions into facts.
   - Stop-and-ask gate: if a requirement depends on an unresolved architectural choice, ask for a decision or present bounded options.

3. **Build the checklist structure**
   - Organize into execution order:
     1. Pre-work/setup
     2. Core implementation
     3. Integration and data/backward compatibility handling
     4. Tests and quality gates
     5. Rollout, monitoring, and backout plan
     6. Documentation and handoff
   - Write checklist items as verifiable actions with clear completion criteria.
   - Include owners/placeholders only when provided or requested.

4. **Expose open questions and risks**
   - Add an explicit "Open Questions" section with blocking vs non-blocking questions.
   - Add a "Risk Controls" section mapping each key risk to mitigation and detection.
   - Stop-and-ask gate: if blockers prevent reliable sequencing, pause and request clarification before finalizing.

5. **Quality-pass the spec**
   - Remove vague verbs (e.g., "improve", "handle") and replace with concrete actions.
   - Ensure each checklist item is testable and mapped to an outcome.
   - Confirm no required report finding was dropped; note intentionally excluded items with reason.

6. **Deliver**
   - Output the final implementation-spec checklist in the style requested by the user (markdown checklist by default).
   - Keep it concise but execution-ready.

## Cursor Implementation Notes

- Prefer reading the report source directly from files/issues/PR context before drafting.
- Use short section headers and markdown checkboxes (`- [ ]`) for default output.
- For codebase-bound plans, include concrete path/symbol hints when known (for example, `src/auth/`, `PaymentService`).
- If the user asks for tickets, split checklist items into independently shippable slices with clear dependencies.

## Safety Rules

- Never invent report facts; label uncertainty explicitly.
- Never hide unresolved blockers inside checklist items; surface them in "Open Questions".
- Never prescribe destructive steps (data deletion, force-push, hard reset) without explicit user approval.
- Never treat compliance/security requirements as optional if the report marks them mandatory.
- Do not claim execution or verification occurred; this skill produces a spec/checklist, not code changes by itself.

## Output Style

When finishing, provide:

1. A one-line objective.
2. An ordered implementation checklist with verifiable items.
3. An "Open Questions" section (blocking first).
4. A "Risk Controls" section (risk -> mitigation -> signal).
5. A short "Out of Scope" section for deferred items.
