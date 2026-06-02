---
name: make-skill
description: Create a new repo-native skill by planning first, declaring assumptions, asking focused clarifying questions, validating the skill name with check-skill-name, and then scaffolding a convention-compliant SKILL.md plus catalog entry. Use when creating a new skill in this repository.
user-invocable: true
disable-model-invocation: true
---

# Make Skill

## Core Contract

Use this skill to create a new skill in this repository, defaulting to `skills/<category>/<name>/SKILL.md` (global). Use `.agents/skills/<name>/SKILL.md` only when the user explicitly asks for a repo-local helper.

Default scope: one new skill at a time, including a compliant `SKILL.md` and a `README.md` catalog entry.

Always start with a short implementation plan and explicit assumptions before authoring files. Then run a focused clarifying-question loop to confirm high-impact decisions.

Treat `CLAUDE.md` / `AGENTS.md` in the target repository as authoritative. If they conflict with this skill, follow them.

## Required Inputs

Gather or infer:

1. The requested skill purpose and primary workflow.
2. Target category under `skills/` (for example `dev`, `code-quality`, `pull-requests`).
3. Proposed skill name, if provided.
4. Any required tools, constraints, and safety rules.
5. Desired output style and verification/reporting expectations.

If critical context is missing, proceed with clearly labeled assumptions and ask only clarifying questions that materially affect behavior, tooling, or safety.

## Workflow

1. **Intake and plan**
   - Restate the requested outcome.
   - Draft a compact implementation plan for the new skill (sections, behavior, and files to touch).
   - Stop-and-ask gate: if the request spans multiple unrelated skills, ask whether to split the work.

2. **State assumptions**
   - List assumptions explicitly (target location defaults to global unless explicitly requested otherwise, category if applicable, invocation defaults, expected level of strictness, output style, and scope boundaries).
   - Mark each assumption as either "safe default" or "needs confirmation."

3. **Run clarifying-question loop**
   - Ask only high-impact questions first, in sequence, not as a large batch.
   - Confirm in/out-of-scope behavior, mandatory tooling, and non-negotiable safety constraints.
   - Do not ask to confirm target location by default; assume `skills/` for repo/content-agnostic global skills.
   - Use `.agents/skills/` only when the user directly requests a repo-opinionated local helper following this repo's `AGENTS.md`.
   - Stop-and-ask gate: if answers materially change behavior, revise assumptions and re-confirm.

4. **Derive and validate the skill name**
   - If the user provided a name, normalize it (lowercase, hyphenated, no spaces/underscores) and validate.
   - If no name is provided, infer from purpose and suggest the shortest reasonable candidate (prefer one word, allow two words when clarity requires).
   - Run `check-skill-name` before scaffolding.
   - If verdict is `CONFLICT` or `RISKY`, propose 3-5 short alternatives and re-check until a `CLEAR` name is chosen.
   - Stop-and-ask gate: do not create files until the name is `CLEAR`.

5. **Author the new skill**
   - Create the `SKILL.md` at the confirmed target location.
   - If target is `skills/`, ensure the content is repository- and content-agnostic.
   - If target is `.agents/skills/`, allow repo-specific conventions and workflows from this repo's `AGENTS.md`.
   - Enforce frontmatter and required section order from repository conventions.
   - Ensure description is third person and ends with "Use when ...".
   - Include explicit stop-and-ask gates in the new skill's workflow.

6. **Update catalog**
   - Add the new skill to the correct section of `README.md` with a concise description.

7. **Verify and report**
   - Verify naming alignment (`name` equals directory name).
   - Verify required sections and policy compliance.
   - Report final outputs and include a short "Assumed vs Confirmed" summary.

## Implementation Notes

- For name validation, invoke `.agents/skills/check-skill-name/SKILL.md` workflow rather than ad-hoc checks.
- Favor defaults for low-impact details to keep momentum; ask questions only where answers change the final skill behavior.
- Keep generated `SKILL.md` concise but complete; avoid unbounded optional sections unless requested.
- Reference existing repo patterns (for example `skills/dev/commit/SKILL.md`) when shaping structure and tone.

## Safety Rules

- Never scaffold files before the skill name is `CLEAR` via `check-skill-name`.
- Never hide assumptions; always label them and confirm high-impact ones.
- Never ask location-clarification by default; assume global unless the user directly requests repo-local.
- Never skip clarifying questions when ambiguity affects behavior, tooling, or safety (excluding default location).
- Never violate required repository conventions for frontmatter and section ordering.
- Never silently broaden scope beyond one requested skill without user approval.

## Output Style

When finishing, report:

1. Chosen category and final skill name (including name-check verdict).
2. Files created/updated.
3. Key assumptions made and which were later confirmed.
4. Any notable defaults applied.
5. Remaining optional improvements the user may choose next.
