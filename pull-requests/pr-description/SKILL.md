---
name: pr-description
description: Refresh a GitHub pull request description so it matches the current branch changeset, including PR discovery or draft PR creation, drift analysis against the PR base branch, concise section rewriting, checklist maintenance, and direct PR body updates through `gh`. Use when the user asks to update, refresh, rewrite, or sync a PR description with the current changes.
---

# PR Description

## Core Contract

Use this skill when the user wants a GitHub PR description brought back in sync with the current branch.

Default to `gh` for all GitHub operations.

This is a private workflow skill. Do not create repo skill files, catalog entries, or command files unless the user explicitly asks.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with them, follow those files.

By default, this skill should update the PR description directly once the new body is ready.

## Required Inputs

Gather or infer:

1. The GitHub PR URL, if the user provided one.
2. Otherwise, the current git branch.
3. The PR associated with that branch on `origin`.
4. The PR base branch.

Do not require the user to paste a PR URL when the current branch can be used to discover the PR.

## Workflow

### 1. Identify the PR

If the user provides a PR URL:

1. Use `gh` to read that PR directly.
2. Confirm the PR is open unless the user explicitly asked to work on a closed PR.

If the user does not provide a PR URL:

1. Inspect the current git branch.
2. Use `gh pr view --json number,title,body,url,headRefName,baseRefName,state,isDraft` from that branch when possible.
3. If that does not return a PR, use `gh pr list --head <branch> --state open --json number,title,body,url,headRefName,baseRefName,state,isDraft`.
4. Treat `origin` as the remote of record for PR discovery.
5. If multiple PRs match the branch, stop and ask the user which PR to use.

If no PR exists for the current branch:

1. Default the draft PR base branch to `develop`.
2. Analyze the branch diff against `develop`.
3. Draft a concise PR title if needed.
   - The title must start with one of the approved prefixes from `.github/workflows/pr-title-check.yml`:
     `proposal:`, `rfc:`, `test:`, `build:`, `ci:`, `cicd:`, `docs:`, `style:`, `fix:`, `perf:`, `refactor:`, `chore:`, `patch:`, `feat:`, `minor:`, `major:`, `breaking:`, or `hotfix:`
4. Create a draft PR with `gh pr create --draft`.
5. Continue the rest of the workflow against the newly created PR.

Stop and ask the user before proceeding if:

- The branch does not match the PR head branch.
- The PR is closed and the user did not ask to reopen or update it anyway.
- More than one open PR matches the branch.
- The repository state makes it ambiguous which branch or PR should be used.

### 2. Read the current PR body and metadata

Collect:

- PR number
- URL
- title
- body
- head branch
- base branch
- draft/open state

Read the full existing PR body before deciding what to keep, rewrite, check off, or remove.

### 3. Analyze the current changeset

Use `git` to understand the branch relative to the PR base branch.

Prefer this sequence:

1. Confirm the base branch from the PR metadata.
2. Fetch enough git context to compare `base...HEAD`.
3. Read the commit history from the divergence point.
4. Read the combined diff from the divergence point.

Minimum analysis:

- `git log --oneline <base>..HEAD`
- `git diff --stat <base>...HEAD`
- `git diff <base>...HEAD`

Use the actual PR base branch when a PR exists.

When creating a new draft PR because none exists yet, compare against `develop`.

Do not rewrite the PR body from commit messages alone. Use the actual diff.

### 4. Detect drift between the PR body and the branch

Compare the existing PR body with the actual branch contents.

Look for:

- Major changes present in the diff but missing from the PR description
- Outdated bullets that no longer reflect the code
- Sections that are still relevant and should be preserved
- Checklist items that can now be checked off
- Checklist items that are stale and should be removed
- Sections that should be omitted because they are no longer relevant

Assume links and attachments are still relevant unless there is strong evidence they are stale.

Preserve all sections that are still relevant, even if the summary content is rewritten.

### 5. Write the updated PR body

The updated PR description must be succinct and easy to scan.

Default structure:

## Summary
- Brief explanation, 1-3 short paragraphs.

## What Changed
- Break this section into short `###` sub-headings
- Put a short bullet list under each sub-heading
- Prefer grouping by concepts, functionality, or application layers
- Avoid a single long flat list for the entire section

Optional sections:

- `## Testing` only when there is relevant testing evidence worth including
- `## Open Questions` only when there are real unresolved questions

Writing rules:

- Use short bullet points
- Do not over-explain
- Prefer outcomes and user-visible or architectural impact over file-by-file churn
- Merge repetitive low-level changes into one practical bullet
- Keep each `What Changed` subsection focused and brief
- Keep the whole body concise

### 6. Preserve and update existing sections

Preserve an existing section when it is still relevant.

Pay special attention to linked issues, closing keywords, and related PR references that already appear in the PR body.

Typical examples:

- linked issues or closing keywords
- related PRs, stacked PRs, dependency PRs, or follow-up PR links
- screenshots or demo links
- rollout or migration notes
- test plans
- checklists

Preserve those links when they are still relevant, even if you rewrite the surrounding prose or reorganize the sections.

When handling checklists:

- Check items off when the current changeset makes completion clear enough to infer confidently
- Leave items unchecked when the diff does not clearly prove completion
- Remove items that are no longer relevant to the current PR scope

Do not preserve stale descriptive bullets just because they already exist. Rewrite summary sections to match the actual diff.
Do not drop linked issues or related PR references unless there is evidence they are stale or no longer relevant to the current PR.

### 7. Update the PR through `gh`

When the new body is ready:

1. Prefer updating the PR body with `gh api --method PATCH repos/<owner>/<repo>/pulls/<number>` rather than `gh pr edit`.
2. Verify the PR body changed as expected by re-reading the PR.

Prefer passing the body as JSON via a temporary file or other quoting-safe mechanism.
Prefer `python3`, not `python`, when generating JSON payload files on machines where only `python3` is available.
Do not inline a large multiline PR body directly into the shell command that performs the PATCH request.
Instead, write the PR body to a temporary markdown file, then use a separate quoting-safe step to serialize it into JSON for `gh api`.

Recommended pattern:

1. Write the new PR body to a temporary `.md` file.
2. Use `python3` to read that file and emit `{"body": ...}` JSON into a temporary `.json` file.
3. Call `gh api --method PATCH ... --input <json-file>`.
4. Re-read the PR with `gh pr view` to verify the body.

Use `gh repo view --json nameWithOwner` if you need to derive `<owner>/<repo>` for the REST call.

Avoid `gh pr edit` for body-only updates in this workflow because it can fail on repos that still expose deprecated Projects (classic) GraphQL fields.

If the PR had to be created in this run, ensure the created draft PR receives the rewritten body rather than a placeholder body.

## GitHub Implementation Notes

Use `gh` for all PR operations.

Useful commands may include:

- `gh pr view`
- `gh pr list --head <branch> --state open`
- `gh pr create --draft`
- `gh api --method PATCH repos/<owner>/<repo>/pulls/<number>`
- `gh api`

For this workflow, prefer `gh api` for PR body updates even when `gh pr edit` would normally seem sufficient.

## Safety Rules

- Never assume the current branch has exactly one PR without verifying it.
- Never rewrite the PR body before reading the existing body.
- Never base the description only on commit titles when the diff is available.
- Never remove links or attachments unless there is evidence they are stale or irrelevant.
- Never check off a checklist item unless the changeset supports that inference with reasonable confidence.
- Never preserve stale descriptive bullets that conflict with the current branch state.
- Never overwrite unrelated user work in the repository while gathering context.
- If unexpected working tree changes appear while you are working, stop and ask the user how to proceed.

## Output Style

When reporting back after updating the PR, include:

1. Which PR was updated or created
2. Which base branch was used for drift analysis
3. The main drift you found between the old description and the current diff
4. The main sections you rewrote, preserved, checked off, or removed
