---
name: changes
description: Inspect the current staged and unstaged working-tree changes and summarize them as logical groupings, using the current branch's history for context and flagging anything unrelated, incidental, or problematic. Use when the user wants to understand or review what they have changed before committing.
---

# Changes

## Core Contract

Use this skill when the user wants a read-only summary of their current working-tree changes, organized into logical groups, with anything out-of-place called out.

Always inspect the actual diff. Read the current branch's history to understand the intended purpose of the work, then judge each change against that intent.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with them, follow those files.

This skill is **read-only**. It does not stage, commit, push, discard, or otherwise modify anything. When the user is ready to commit, hand off to the `commit` skill.

## Required Inputs

Gather or infer:

1. The current git branch and its base (`main`/`develop` or other default).
2. Scope: all changes, or specific paths the user named.
3. The intent of the branch — from its commit history, name, and any linked PR.
4. The level of detail wanted (quick overview vs. per-group breakdown). Default: per-group breakdown.

## Workflow

### 1. Capture the current state

1. Run `git status --porcelain` to enumerate staged, unstaged, and untracked files.
2. Run `git diff` (unstaged) and `git diff --staged` to see the actual content.
3. Note untracked files separately — they are part of the change set but invisible to `git diff`.
4. If the working tree is clean, report that there is nothing to summarize and stop.

### 2. Establish branch context

1. Identify the base branch and run `git log --oneline <base>..HEAD` to see what this branch already does.
2. Read the branch name and, if present, the PR title/description for stated intent.
3. Form a one-line hypothesis of the branch's purpose. The working-tree changes should extend that purpose; deviations are what you flag.

### 3. Group the changes logically

- Cluster files and hunks by the unit of work they serve (feature, fix, refactor, test, config, docs), not by directory.
- A single file may contribute hunks to more than one group; split at the hunk level when that is clearer.
- Name each group by its purpose, list the files/hunks in it, and describe what it does in one or two sentences.

### 4. Flag what is unrelated or problematic

Call out, per item, with `file:line` references:

- **Unrelated** — changes that do not serve the branch's stated purpose (a drive-by fix, an unrelated refactor, a stray formatting sweep).
- **Incidental** — debug prints, commented-out code, `TODO`/`FIXME`, leftover scaffolding, version bumps, whitespace-only churn.
- **Risky** — secrets, credentials, `.env` values, hardcoded tokens, large or binary artifacts, generated files checked in by hand.
- **Inconsistent** — changes that contradict the branch history, partially-applied edits, or a change made in one place but missed in a sibling.

For each, say *why* it stands out and suggest whether it belongs in this change, a separate commit, or should be reverted. Do not act on the suggestion — surface it.

### 5. Report

Present the grouped summary followed by the flagged items. Offer the `commit` skill as the next step if the user wants to proceed.

## Implementation Notes

- Useful commands: `git status --porcelain`, `git diff`, `git diff --staged`, `git diff --stat`, `git log --oneline <base>..HEAD`, `git diff <base>...HEAD` for the full branch delta.
- Untracked files do not appear in `git diff`; read them directly to judge their content.
- Detect the base branch from the repo default (`git remote show origin`, or `gh repo view --json defaultBranchRef`) or a `develop` convention.
- For a large change set, lead with `git diff --stat` to scope groups before reading full hunks.

## Safety Rules

- Never stage, commit, push, amend, stash, reset, or discard. This skill only reads and reports.
- Never run `git add` — not even to inspect.
- Do not edit files to "clean up" what you flag; report it and let the user decide.
- If you spot a secret or credential, flag it prominently and do not echo its value in full.

## Output Style

When finishing, report:

1. The branch and its base, plus the one-line purpose you inferred.
2. The logical groups — each with a name, its files/hunks, and what it does.
3. Flagged items — unrelated, incidental, risky, or inconsistent — with `file:line` and why.
4. A short bottom line: does the change set look cohesive, or does it need splitting before commit.
