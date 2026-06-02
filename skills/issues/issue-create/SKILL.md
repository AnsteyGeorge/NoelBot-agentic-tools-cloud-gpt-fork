---
name: issue-create
description: Create a GitHub issue by normalizing required fields, deriving missing context from branch PR metadata via pr-info when needed, applying optional metadata, and verifying the created issue state. Parent issue linking is optional, and when requested this skill creates a real GitHub sub-issue relationship with explicit verification and ambiguity gates. Use when a user asks to create an issue and optionally attach it under a parent issue.
user-invocable: true
disable-model-invocation: false
---

# Issue Create

## Core Contract

Use this skill to create a new GitHub issue safely and consistently from structured or
free-form context. The skill must produce a created issue URL and a verified final state.

Default tool is `gh`. Resolve repository context from the current git remote unless the
user provides an explicit repository.

If a parent issue is provided, this skill must create a real GitHub parent-child
relationship (sub-issue link) after issue creation and verify that relationship is present.
Do not replace this with body text references unless the user explicitly changes scope.

If issue title/body intent is missing or too thin, call `pr-info` to load branch/PR context
first, then derive a draft issue payload from that context before asking follow-up questions.

Parent issue input is optional. Standalone issue creation is valid when no parent context is
provided.

When called from `issue-link-pr` fallback flow, parent selection order is:
1. best matching branch issue that has children,
2. root issue as final fallback.

Treat `CLAUDE.md` / `AGENTS.md` in the target repository as authoritative and follow
them on conflict.

## Required Inputs

Gather or infer:

1. Target repository (`owner/repo`) for issue creation.
2. Issue context:
   - required at create time: title and body intent,
   - optional: labels, assignees, milestone, project references.
3. Context derivation sources when direct issue context is missing:
   - PR metadata from `pr-info` (`number`, `title`, `body`, `url`, branch/base intent),
   - branch intent inferred from the resolved PR.
4. Parent issue reference (optional): number or URL.
5. Visibility and access constraints for repository, parent issue, and metadata targets.
6. Fallback-parent context when invoked by matching flow (optional but expected there):
   - root issue reference,
   - best matching branch issue reference (must have children),
   - reason no strong leaf match existed.

If title or body intent is missing, run `pr-info` first to derive them. Stop and ask only if
the payload remains insufficient or ambiguous after derivation.

## Workflow

1. **Acquire issue context**
   - If user supplied clear title and body intent, keep them as authority.
   - If context is missing or weak, run `pr-info` and derive a draft issue title/body intent
     from PR title, PR body, and branch intent.
   - Preserve user wording and user-provided constraints while deriving.
   - Stop and ask if context is still insufficient after `pr-info`.

2. **Resolve repository and permissions**
   - Infer repository from current git remote when not provided.
   - Verify the actor can create issues in the target repository.
   - Stop and ask on missing repository context or insufficient permissions.

3. **Normalize issue payload**
   - Build a clear title from authoritative context (user input first, then derived PR context).
   - Draft body with concise problem/context/acceptance details.
   - Normalize optional metadata (labels, assignees, milestone) to canonical names.
   - Stop and ask if required metadata is invalid and cannot be mapped confidently.

4. **Create the issue**
   - Create with `gh issue create` using title/body and any valid optional fields.
   - Capture created issue number and URL immediately.
   - Stop and ask on create failures rather than retrying with broadened assumptions.

5. **Select and verify parent when parenting is requested**
   - If no parent issue or fallback-parent context is provided, skip to final verification.
   - If caller passed explicit parent issue, use it after verification.
   - If caller passed fallback-parent context, select parent in this order:
     1. best matching branch issue that has children,
     2. root issue.
   - Verify selected parent exists, is accessible, and is valid for sub-issue linking.
   - Stop and ask if fallback-parent context is incomplete for fallback mode.
   - Stop and ask if parent selection cannot be made deterministically.

6. **Attach to parent when selected**
   - Create a real sub-issue relationship using GitHub GraphQL mutation.
   - Stop and ask if the parent is ineligible, inaccessible, cross-repo unsupported, or
     if GraphQL returns policy/permission errors.

7. **Verify final state**
   - Re-read the created issue and confirm title/body/metadata match request.
   - If parent linking was requested, verify parent-child relationship exists from API response.
   - Report final issue URL, context source, parent linkage status, and any metadata not applied.

## GitHub Implementation Notes

- Create issue:
  - `gh issue create --title "<title>" --body-file <file> [--label ...] [--assignee ...] [--milestone ...]`
  - Read back with `gh issue view <number-or-url> --json number,title,body,url,labels,assignees,milestone,state,repository`
- Resolve parent issue:
  - `gh issue view <number-or-url> --json id,number,title,url,state,repository`
  - For best-branch eligibility, verify parent has children:
    - `gh api graphql -f query='query($owner:String!,$repo:String!,$number:Int!){repository(owner:$owner,name:$repo){issue(number:$number){number url subIssues(first:1){nodes{number}}}}}' -F owner=<owner> -F repo=<repo> -F number=<number>`
- Create real sub-issue link (GraphQL IDs required):
  - `gh api graphql -f query='mutation($issueId:ID!,$subIssueId:ID!){addSubIssue(input:{issueId:$issueId,subIssueId:$subIssueId}){issue{id number url}subIssue{id number url}}}' -F issueId=<parentNodeId> -F subIssueId=<childNodeId>`
- Verify linkage by querying parent `subIssues` or child `parent` relation:
  - `gh api graphql -f query='query($owner:String!,$repo:String!,$number:Int!){repository(owner:$owner,name:$repo){issue(number:$number){number url subIssues(first:100){nodes{number url}}}}}' -F owner=<owner> -F repo=<repo> -F number=<parentNumber>`
- Prefer deterministic, minimal issue bodies; place machine-added context in one short
  section rather than rewriting user-authored prose.

## Safety Rules

- Never create an issue without confirming minimally sufficient title/body intent.
- Never create a placeholder issue from empty context before running `pr-info`.
- Never claim parent linkage succeeded without API-level verification.
- Never silently downgrade to a text-only parent reference when real linkage fails.
- Never invent labels, assignees, milestones, or parent IDs; validate each explicitly.
- Never violate fallback parent ordering in matching fallback mode.
- Never use a best-branch fallback parent that has no child issues.
- Stop and ask on permission errors, ambiguous repository resolution, or parent mismatch.

## Output Style

When finishing, report:

1. Created issue (number, title, URL, repository).
2. Context source summary (user-provided, `pr-info` derived, or mixed).
3. Applied metadata (labels, assignees, milestone) and anything skipped.
4. Parent selection summary when parent linking was requested:
   - explicit parent or fallback mode,
   - selected parent (best branch or root),
   - reason the selected parent was chosen.
5. Parent linkage result: linked as sub-issue or not linked (with reason).
6. Verification checks performed and outcomes.
7. Any stop-and-ask gates encountered and how they were resolved.
