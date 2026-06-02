---
name: build
description: Implement a scoped change from provided context, a plan, or a speclist by converting inputs into concrete tasks, applying code edits, validating results, and reporting outcomes with assumptions and follow-ups. Use when the user wants execution started from existing context or planning artifacts.
user-invocable: true
disable-model-invocation: false
---

# Build

## Core Contract

Use this skill when the user wants implementation, not just analysis, and has provided either free-form context, an explicit plan, a speclist-style checklist, or a filepath to one of those artifacts.

Default scope: execute one cohesive change end-to-end in the current repository, including edits, validation, and a concise delivery report.

Treat `CLAUDE.md` / `AGENTS.md` in the target repository as authoritative. If they conflict with this skill, follow them.

## Required Inputs

Gather or infer:

1. Execution source: `context`, `plan`, or `speclist`.
   - May be inline content or a filepath to a markdown artifact.
2. Target outcome and scope boundaries (what should change, what must not change).
3. Required validations (tests, lint, typecheck, smoke checks) and acceptable evidence.
4. Delivery expectations (code only, code + docs, commit/PR handling if requested).
5. Known constraints (deadline, risk tolerance, protected branches, tooling limits).

If no actionable implementation target can be extracted, stop and ask for clearer scope before editing files.

## Workflow

1. **Classify and normalize input**
   - Identify whether the user provided `context`, `plan`, or `speclist`, including when provided as a filepath.
   - If a filepath is provided, read and classify the file contents before execution.
   - Convert it into an execution brief: objective, constraints, deliverables, validation.
   - Stop-and-ask gate: if the input contains multiple unrelated initiatives, ask whether to split and which one to execute first.

2. **Check readiness for implementation**
   - If input is `context` and still ambiguous, derive a compact task list.
   - If input is `plan`, turn plan steps into concrete file/symbol changes.
   - If input is `speclist`, execute checklist items in order and keep dependencies explicit.
   - When executing from a speclist file, check off items only after corresponding work is implemented and validated.
   - If implementation scope changes, edits to the speclist are permitted (for example, splitting tasks, reordering, adding clarified items) as long as the updated list remains truthful to current scope.
   - Stop-and-ask gate: if a high-impact architectural decision is unresolved, ask for a decision or present bounded options.

3. **Prepare execution**
   - Inspect relevant files and existing patterns before editing.
   - Define a minimal implementation sequence and validation sequence.
   - Call out assumptions explicitly and proceed when they are safe defaults.

4. **Implement incrementally**
   - Apply focused edits in small, reviewable chunks.
   - Keep behavior aligned with repository conventions and existing abstractions.
   - Add brief comments only where logic is non-obvious.
   - If unexpected unrelated working-tree changes appear, pause and ask how to proceed.

5. **Validate**
   - Run the narrowest meaningful checks first, then broader checks as needed.
   - If validation fails, fix root causes and re-run until green or blocked.
   - For speclist files, only mark checklist items complete after validation evidence exists for that item or its acceptance criteria.
   - Stop-and-ask gate: if external systems, secrets, or missing permissions block verification, report exactly what is needed and wait.

6. **Deliver**
   - Summarize what changed, why, and how it was validated.
   - List assumptions, residual risks, and optional next steps.
   - If commit/PR actions were requested, hand off to the appropriate workflow and include verification status.

## Implementation Notes

- Prefer direct file/tool evidence over speculation; read before editing.
- Use `rg`/`Glob` to find symbols and paths quickly, then `ReadFile` for context.
- After substantive edits, run lints on touched files when available.
- Keep command usage minimal and purposeful; avoid broad destructive operations.
- When the execution source is under-specified, default to the smallest safe implementation slice and report what remains.
- When the execution source is a speclist filepath, keep the file updated as live execution state.

## Safety Rules

- Never treat ambiguous intent as approval for broad refactors.
- Never claim validation ran when it did not; distinguish executed checks from proposed checks.
- Never commit, push, or open a PR unless the user asked.
- Never revert or overwrite unrelated user changes without explicit instruction.
- Never use destructive git/file operations without explicit approval.

## Output Style

When finishing, report:

1. The execution source used (`context`, `plan`, or `speclist`) and the interpreted objective.
2. Source filepath used (if any), and whether the speclist file was updated.
3. Files changed and the key implementation decisions.
4. Validation run and outcomes (or exact blockers).
5. Assumptions made and any unresolved questions.
6. Optional next steps (tests, commit, PR, rollout) when relevant.
