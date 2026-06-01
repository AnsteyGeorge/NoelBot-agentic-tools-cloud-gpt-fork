---
name: pr-ci
description: Investigate and fix failed GitHub pull request CI jobs for the current branch, including PR discovery, branch verification, failed-job log review, failure deduplication, root-cause investigation, code changes, commit, and push. Use when the user wants CI failures for the current branch handled end-to-end.
---

# PR CI

## Core Contract

Use this skill when the user wants the failed CI for the current branch's GitHub PR investigated and fixed end-to-end.

Default to `gh` for all GitHub operations.

This is a private workflow skill. Do not create repo skill files, catalog entries, or command files unless the user explicitly asks.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with them, follow those files.

By default, this is an end-to-end workflow: investigate, fix, validate, commit, and push unless the user explicitly scopes it down.

## Required Inputs

Gather or infer:

1. The current git branch.
2. The PR associated with that branch.
3. Whether the user wants the full end-to-end flow if they did not already say so.

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

### 2. Read finished CI results for the PR

Inspect the PR checks and completed workflow runs.

Use `gh pr checks <pr-number>` first for a quick view, then use `gh run list` or `gh run view` as needed to inspect the finished runs tied to the PR branch.

Collect, for every finished failed job:

- Workflow name
- Job name
- Run id
- Run URL if available
- Commit SHA
- Failure conclusion (`failure`, `timed_out`, `cancelled`, etc.)
- Whether it looks code-related, infra-related, or unclear

Ignore jobs that are still pending or in progress until they finish.

### 3. Read failure output for every failed job

For each failed job:

1. Fetch the detailed job output with `gh run view <run-id> --log` or the closest equivalent that exposes the failing step output.
2. Capture the failing step name.
3. Capture the most relevant error lines, stack traces, or test failures.
4. Note whether the failure is likely duplicated across multiple jobs.

Do not stop at the status summary. Read enough log output to understand the actual failure.

### 4. Deduplicate failures by root cause

Different jobs may fail for the same reason.

Group failed jobs by unique failure cause, for example:

- The same TypeScript error repeated across matrix jobs
- The same lint violation repeated in multiple shards
- A single failing test causing multiple package jobs to fail
- A shared setup problem breaking several workflows

For each unique failure, keep:

- The representative job or jobs
- The exact error evidence
- Your current hypothesis for the root cause

### 5. Investigate each unique failure

For each unique failure:

1. Read the relevant code, config, tests, or scripts referenced by the logs.
2. Confirm whether the failure is:
   - A real code issue that should be fixed
   - Already fixed on the branch and only needs a rerun
   - A flaky test or nondeterministic failure
   - An external or infrastructure issue that code changes should not attempt to paper over
   - Unclear and needs user input
3. Trace back to the smallest real root cause rather than fixing repeated symptoms.

Do not guess from the job name alone. Use the log output and source code together.

Ask the user before proceeding if the fix would require:

- A product or architecture decision
- A risky refactor beyond the CI failure itself
- A change that conflicts with existing instructions
- Choosing between multiple plausible implementations with meaningful trade-offs

### 6. Fix each confirmed code issue

When the root cause is a real code problem:

1. Make the smallest correct change that addresses the actual cause.
2. Keep the change scoped unless a nearby fix is clearly necessary.
3. Follow existing project conventions and patterns.
4. Update tests only when they materially reduce regression risk or when nearby coverage should move with the change.

When the root cause is already fixed by current branch state:

- Do not make a no-op code change.
- Validate locally and plan to rerun or report that no new code change is needed.

When the failure is flaky or external:

- Do not invent code changes just to get green CI.
- Gather enough evidence to explain why it is flaky or external.
- Ask the user whether they want a rerun or a follow-up investigation if a code fix is not justified.

### 7. Run validation before committing

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

### 8. Commit the fix

If you made code changes and validation passes:

1. Stage the relevant files.
2. Create a single commit for the CI-fix work.

Commit message rules:

- One line only
- Very concise
- No prefixes
- No shortcode
- No trailing period
- Describe the actual code change, not the fact that CI was fixed
- Be specific enough that someone can understand what changed from the commit subject alone

Good examples:

- `Register method not allowed plugin`
- `Export service params helpers`
- `Wrap IAM services with traceService`

Bad examples:

- `fix: address CI failures`
- `[ci] fix build`
- `Fix PR CI`
- `Address CI failures`
- `Address CI failures.`

If no code changes were needed because the failures were already fixed, flaky, or external, do not create an empty commit.

### 9. Push the branch

After a successful commit, push the current branch to its remote.

Use a normal push. Never force-push unless the user explicitly asks.

If there was no new commit, do not push unless another branch-state reason makes a push necessary.

### 10. Summarize remaining CI actions

After the code is fixed and pushed:

- Note which failed jobs should pass on the next run.
- Note any jobs that appear flaky or externally blocked.
- If appropriate, suggest rerunning CI or waiting for GitHub to rerun automatically.

Do not claim CI is fixed unless you have evidence that the root causes were addressed.

## GitHub Implementation Notes

Use `gh` for PR, checks, and workflow-run operations.

Recommended sequence:

1. Discover the PR from the current branch and read its metadata.
2. Read PR checks.
3. Enumerate failed completed runs and jobs.
4. Read logs for every failed job.
5. Group failures by unique cause.
6. Investigate source-level root causes.
7. Make code changes locally.
8. Validate.
9. Commit.
10. Push.

Useful commands may include:

- `gh pr view --json ...`
- `gh pr list --head <branch> --state open --json ...`
- `gh pr checks <pr-number>`
- `gh run list --branch <branch>`
- `gh run view <run-id>`
- `gh run view <run-id> --log`

When a direct `gh` subcommand is insufficient, use `gh api`.

## Safety Rules

- Never assume the current branch has a unique open PR; discover and verify it.
- Never rely only on check summaries when logs are available.
- Never treat duplicate failed jobs as separate root causes without evidence.
- Never create an empty commit just to mark CI progress.
- Never push if validation is failing.
- Never force-push unless explicitly requested.
- Never overwrite or revert unrelated user changes.
- Never fabricate code changes for flaky or infrastructure failures.
- If unexpected changes appear while working, stop and ask the user how to proceed.

## Output Style

When finishing the task, report:

1. Which failed jobs were identified
2. Which unique failure groups were found
3. The root cause for each unique failure
4. Which issues were fixed with code changes
5. Which failures were already fixed, flaky, or external
6. The commit message used
7. Whether the branch was pushed
8. Any remaining blockers or rerun recommendations
