---
name: pr-comments
description: Address unresolved GitHub pull request review comments for the current branch, including PR discovery, branch verification, comment triage, code changes, commit, push, and comment resolution. Use when the user wants review feedback for the current branch handled end-to-end.
---

# Address PR Comments

## Core Contract

Use this skill when the user wants the unresolved review feedback on the current branch's GitHub PR addressed.

Default to `gh` for all GitHub operations.

This is a private workflow skill. Do not create repo skill files, catalog entries, or command files unless the user explicitly asks.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with them, follow those files.

## Required Inputs

Gather or infer:

1. The current git branch.
2. The PR associated with that branch.
3. Whether the user wants you to actually commit and push after making changes.

Do not require the user to paste a PR link when the branch can be used to discover it.

## Workflow

### 1. Discover the PR for the current branch

Use the **`pr-info`** skill to resolve and verify the PR for the current branch — invoke it
via the Skill tool where supported, otherwise follow its workflow. It returns the PR
`number`, `headRefName`, `baseRefName`, `state`, `isDraft`, and `url`, and applies the
standard gates (no PR, multiple PRs, branch mismatch, closed/merged). Do not re-implement PR
discovery here.

In addition, before making edits, stop and ask the user if the working tree has unrelated
user changes that would make safe edits or commits ambiguous.

If `pr-info` returns a single matching PR, continue on that branch.

### 2. Read all unresolved review threads

Use `gh` to fetch the review discussion for the PR.

Prefer GraphQL so you can inspect review threads directly, including whether each thread is resolved and what file/line it refers to.

Collect, for every unresolved thread:

- Thread id
- File path
- Line or line range if available
- Original comment author
- Full comment text
- Any existing replies in the thread

Ignore already resolved threads unless a new unresolved reply re-opened the discussion.

### 3. Triage each unresolved thread

For each unresolved thread:

1. Read the relevant code and nearby context.
2. Understand whether the feedback is:
   - Correct and should be implemented
   - Reasonable but already addressed by current branch state
   - Intentional non-action because the suggestion is incorrect, harmful, redundant, or outside scope
   - Unclear and needs user input
3. Check whether the thread points to outdated code and whether the current diff already makes it obsolete.

Do not blindly apply every suggestion. Evaluate correctness, risk, consistency with project rules, and whether the change improves the branch.

Ask the user before proceeding if a comment implies:

- A product or architecture decision
- A change that conflicts with existing instructions
- Multiple plausible implementations with meaningful trade-offs
- A risky refactor beyond the scope of the review feedback

### 4. Make worthwhile code changes

When a comment is worth addressing:

1. Edit the code on the current branch.
2. Keep the change scoped to the feedback unless a nearby fix is clearly necessary.
3. Follow existing project conventions and patterns.
4. Update tests only when they materially reduce regression risk or when nearby coverage should move with the change.

When a comment is already satisfied by the current branch state:

- Do not make a no-op code change just to satisfy the thread.
- Plan to reply concisely or resolve it after verifying the branch already handles the concern.

When intentionally not making a suggested change:

- Prepare a short, plain-language explanation.
- Keep it factual and concise.
- Do not argue at length.

### 5. Run validation before committing

After substantive edits, run quality checks appropriate to the touched code.

Minimum expectations:

1. Lint changed areas.
2. Typecheck each changed package.
3. Run targeted tests when the changed code has meaningful test coverage or regression risk.

In this repo:

- Run `pnpm lint` after substantial changes.
- Prefer focused typechecks with `pnpm exec tsc --build --noEmit -f <path/to/tsconfig.json>`.
- Run targeted tests for touched packages when relevant.

Do not commit if validation fails. Fix obvious issues first; otherwise stop and report the blocker to the user.

### 6. Commit the addressed changes

If you made code changes and validation passes:

1. Stage the relevant files.
2. Create a single commit for the review-addressing work.

Commit message rules:

- One line only
- Very concise
- No prefixes
- No shortcode
- No trailing period
- Describe the actual code change, not the fact that you addressed review comments
- Be specific enough that someone can understand what changed from the commit subject alone

Good examples:

- `Register method not allowed plugin`
- `Export service params helpers`
- `Wrap IAM services with traceService`

Bad examples:

- `fix: address PR comments`
- `[review] address comments`
- `Address review feedback`
- `Handle PR comments`
- `Address review feedback.`

If no code changes were needed for any unresolved thread, do not create an empty commit.

### 7. Push the branch

After a successful commit, push the current branch to its remote.

Use a normal push. Never force-push unless the user explicitly asks.

If there was no new commit because all unresolved threads were already effectively addressed, skip pushing unless another push is still required for branch state.

### 8. Resolve or reply to every unresolved thread

Only do this after the final branch state is ready.

For each previously unresolved thread:

- If you addressed it with a code change, resolve the thread.
- If the current branch already addressed it and the thread is now outdated, resolve the thread.
- If you intentionally did not make the suggested change, post a simple concise reply explaining why, then resolve the thread.

Reply style:

- Plain language
- Short
- Specific
- No defensiveness

Examples:

- `Handled in the latest commit by moving this into the shared helper.`
- `This is already covered by the current branch state, so I’m resolving this thread.`
- `Keeping this as-is because the service needs to preserve the existing API contract.`

If a thread is still genuinely unresolved after investigation, do not resolve it. Summarize the blocker and ask the user how to proceed.

## GitHub Implementation Notes

Use `gh` for PR and review-thread operations.

Recommended sequence:

1. Discover the PR from the current branch and read its metadata.
2. Fetch unresolved review threads with GraphQL.
3. Make code changes locally.
4. Validate.
5. Commit.
6. Push.
7. Reply and resolve threads through `gh`.

When a direct `gh` subcommand is insufficient, use `gh api graphql`.

## Safety Rules

- Never assume the current branch has a unique open PR; discover and verify it.
- Never resolve comments before the branch state is finalized.
- Never create an empty commit just to mark review progress.
- Never push or resolve threads if validation is failing.
- Never force-push unless explicitly requested.
- Never overwrite or revert unrelated user changes.
- If unexpected changes appear while working, stop and ask the user how to proceed.

## Output Style

When finishing the task, report:

1. Which unresolved threads were addressed with code changes
2. Which were resolved without code changes
3. Which received an explanatory reply instead of a code change
4. The commit message used
5. Whether the branch was pushed
6. Any threads left unresolved and why
