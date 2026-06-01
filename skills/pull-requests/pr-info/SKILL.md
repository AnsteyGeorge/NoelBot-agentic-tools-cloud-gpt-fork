---
name: pr-info
description: Discover and load the GitHub pull request for the current branch (or a provided PR URL), verify it unambiguously matches the working branch, and return its core metadata for other workflows to build on. Covers branch inspection, gh-based PR lookup with a list fallback, and the standard stop-and-ask gates for missing, duplicate, mismatched, or closed PRs. Use when a workflow needs to identify and load PR context before acting, or when the user asks which PR corresponds to the current branch.
---

# PR Info

## Core Contract

Use this skill to resolve the **single** GitHub PR that the current work applies to and
load its context. It is the shared front-door step for the other `pr-*` skills — they call
it first, then act on the metadata it returns. It is read-only: it discovers, verifies, and
reports; it never edits code, the PR, or git state.

Default to `gh` for all GitHub operations. Treat `origin` as the remote of record.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with
them, follow those files.

## Required Inputs

Gather or infer:

1. A PR URL or number, if the user provided one.
2. Otherwise, the current git branch (`git branch --show-current`).

Do not require the user to paste a PR link when the current branch can be used to discover it.

## Workflow

### 1. Resolve the PR

If the user provided a PR URL or number:

1. Read it directly with `gh pr view <url-or-number> --json <fields>`.

Otherwise, discover from the current branch:

1. Inspect the current git branch.
2. Try `gh pr view --json <fields>` from that branch.
3. If that returns no PR, fall back to
   `gh pr list --head <branch> --state open --json <fields>`.

Use this field set so callers have everything they typically need:

```
number,title,body,url,headRefName,baseRefName,state,isDraft,author,mergeStateStatus
```

### 2. Verify and gate

Once a candidate PR is found, compare its `headRefName` to the current branch.

**Stop and ask the user** before returning if any of these hold:

- No PR exists for the current branch (a caller such as `pr-description` may then choose to
  create a draft — that is the caller's decision, not this skill's).
- More than one open PR matches the branch.
- The current branch does not match the PR `headRefName`.
- The PR is closed or merged and the user did not ask to work on it anyway.

Only return a PR when exactly one open PR unambiguously matches the current branch (or the
user explicitly named one).

### 3. Return the context

Report the resolved PR's metadata so the calling workflow can proceed without re-querying:
`number`, `title`, `url`, `headRefName`, `baseRefName`, `state`, `isDraft`, and the `body`
when intent matters (e.g. for conflict resolution or description drift). Treat the PR
title/body as the authority for *what the branch is trying to accomplish*.

## GitHub Implementation Notes

Useful commands:

- `gh pr view --json number,title,body,url,headRefName,baseRefName,state,isDraft,author,mergeStateStatus`
- `gh pr view <url-or-number> --json <fields>`
- `gh pr list --head <branch> --state open --json <fields>`
- `gh repo view --json nameWithOwner` when a caller needs `<owner>/<repo>` for REST calls.

When a direct `gh` subcommand is insufficient, use `gh api` / `gh api graphql`.

## Safety Rules

- Never assume the current branch has exactly one open PR — discover and verify it.
- Never return a PR whose head branch does not match the current branch without flagging it.
- Never edit code, git state, or the PR; this skill only reads and reports.
- If unexpected repository or branch state appears, stop and ask the user how to proceed.

## Output Style

When finishing, report:

1. How the PR was resolved (current branch vs. user-provided URL).
2. The PR number, title, and URL.
3. The head and base branches, and the open/draft state.
4. Any gate that required a user decision (no PR, multiple PRs, mismatch, closed/merged).
