---
name: commit
description: Create one or more well-formed git commits from the current working-tree changes, including reviewing the diff, grouping related changes into logical commits, writing clear messages in the repo's convention, and running pre-commit hooks. Use when the user wants their current changes committed.
---

# Commit

## Core Contract

Use this skill when the user wants the current working-tree changes turned into one or more well-formed commits.

Always inspect the actual diff before committing. Match the repository's existing commit conventions.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with them, follow those files.

By default this stages and commits only. It does not push unless the user asks.

## Required Inputs

Gather or infer:

1. The current git branch (and whether it is a protected base like `main`/`develop`).
2. Scope: all current changes, or specific paths.
3. Whether the changes form one logical commit or several.
4. The commit message — use the user's wording if given, otherwise infer it from the diff.
5. Whether to push afterward (default: no).

## Workflow

### 1. Inspect the working tree

1. Start by running the `changes` skill to get a read-only summary of the working tree: its logical groupings and any unrelated, incidental, risky, or inconsistent items, judged against the branch's history. Use that as the basis for grouping below.
2. Run `git log --oneline -15` to learn the repo's message style and conventions.
3. Identify the current branch. If it is `main`, `develop`, or another protected base, stop and confirm before committing.
4. If the working tree is clean, report that there is nothing to commit and stop.

### 2. Decide commit grouping

- Build on the groups `changes` already identified.
- If all changes serve one cohesive purpose, plan a single commit.
- If unrelated changes are mixed together, propose splitting them into logical commits and stage each subset separately (pathspecs or `git add -p`).
- Do not sweep in the unrelated or incidental edits `changes` flagged. Surface them to the user instead.

### 3. Stage deliberately

1. Stage exactly the files or hunks for each planned commit.
2. Avoid a blind `git add -A` / `git add .` without reviewing what it includes.
3. Re-check `git diff --staged` before committing.

### 4. Write the message

- Follow the convention observed in `git log` and any rule in `CLAUDE.md` / `AGENTS.md` (e.g. Conventional Commits, required trailers).
- Absent a clear convention: a concise imperative subject (~50 chars), plus a body explaining *why* when the change is non-trivial.
- Describe the change itself, not the act of committing.

### 5. Run pre-commit hooks

- Let hooks run. If a hook reformats files, re-stage the result and proceed.
- If a hook fails, read its output, fix the underlying cause, and retry.
- Do not bypass hooks with `--no-verify` unless the user explicitly asks.

### 6. Commit

- Create each planned commit. For multiple commits, repeat staging + message per group.

### 7. Report

- Show the commit(s) created. Push only if the user requested it.

## Implementation Notes

- Useful commands: `git status --porcelain`, `git diff`, `git diff --staged`, `git add -p`, `git add <pathspec>`, `git commit`, `git log --oneline`.
- Detect a protected base from the repo default branch (`git remote show origin`, or `gh repo view --json defaultBranchRef`) or a `develop` convention.
- Before staging, scan the diff for secrets, credentials, `.env` values, and large or binary artifacts.

## Safety Rules

- Never commit secrets, credentials, or large build artifacts; scan the diff first.
- Never blind-stage everything without reviewing what is included.
- Never amend or rewrite already-pushed commits unless the user asks.
- Confirm before committing on `main`, `develop`, or another protected base.
- Do not bypass failing hooks with `--no-verify` unless explicitly told.
- Do not push unless the user asks.
- If unexpected or unrelated changes appear, stop and ask the user how to proceed.

## Output Style

When finishing, report:

1. The branch committed on.
2. The commit(s) created (short hash + subject).
3. How the changes were grouped.
4. Any hook actions taken (reformat, fixes).
5. Anything intentionally left uncommitted.
6. Whether the branch was pushed.
