---
name: conflicts
description: Resolve an in-progress git conflict state (merge, rebase, cherry-pick, or revert) by resolving each conflicted file using branch intent and correct upstream integration, then continuing the operation to completion. Use when a git operation has stopped with conflicts that need resolving.
---

# Conflicts

## Core Contract

Use this skill when a git operation has stopped with conflicts to resolve. It resolves an **already in-progress** operation; it does not start one. It is the shared conflict-resolution skill that the `rebase` skill hands off to.

Resolve using the branch's intent plus correct integration with upstream. Never blindly pick one side.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with them, follow those files.

## Required Inputs

Gather or infer:

1. Which operation is in progress (merge, rebase, cherry-pick, or revert).
2. The branch's intent: its own recent commits, the PR title/body if available, and any user-stated goal.

## Workflow

### 1. Detect the operation state

1. Run `git status` to see the conflict summary and operation hints.
2. Distinguish the operation from repo state: `.git/MERGE_HEAD` (merge), `.git/rebase-merge` or `.git/rebase-apply` (rebase), `.git/CHERRY_PICK_HEAD` (cherry-pick), `.git/REVERT_HEAD` (revert).
3. If no operation is in progress, report that there is nothing to resolve and stop.

### 2. Establish branch intent

Summarize what this branch is trying to deliver — its own commits (`git log`), and `gh pr view --json title,body` if a PR exists. Treat this as the authority when choosing resolutions.

### 3. List conflicts

Use `git diff --name-only --diff-filter=U` for conflicted paths. Also note add/add, delete/modify, and rename conflicts, which need special handling.

### 4. Resolve each file in context

For each conflicted file:

1. Read both sides and the surrounding code, not only the conflict hunk. Understand what each side means.
2. Choose a resolution aligned with, in priority order: **branch intent / PR description** → **recent branch commits** → **minimal-correct integration with upstream** (e.g. take upstream refactors while preserving the branch's behavior).
3. For delete/modify, decide whether the file should survive; for add/add, merge both additions coherently.
4. Remove all conflict markers and ensure the file is coherent (imports, types, syntax, formatting).
5. `git add` the resolved path.

When a resolution is still ambiguous after using intent and upstream context, stop and ask the user with a concrete question (file + options). Do not guess.

### 5. Validate

Sanity-check the resolved files. Run a quick lint/typecheck/build on touched areas when readily available. Grep for leftover markers (`<<<<<<<`, `=======`, `>>>>>>>`) before continuing.

### 6. Continue the operation

Run the correct continuation for the detected operation:

- rebase: `git rebase --continue` (repeat resolve → continue for each step that conflicts).
- merge: stage all resolved files, then `git commit` (or `git merge --continue`).
- cherry-pick: `git cherry-pick --continue`.
- revert: `git revert --continue`.

For multi-commit rebases, loop until the operation finishes.

### 7. Report

Summarize the resolutions and the final state. Do not push unless asked — the caller (or the `rebase` skill) handles pushing.

## Implementation Notes

- Useful commands: `git status`, `git diff --name-only --diff-filter=U`, `git ls-files -u`, `git checkout --ours/--theirs <path>` (only when taking a whole side is correct), `git add`, and the per-operation `--continue` commands.
- Only run `--abort` if the user asks.

## Safety Rules

- Never blindly accept one side; resolve by meaning and intent.
- Ask the user when a resolution is genuinely ambiguous after using available context.
- Never leave conflict markers in committed files.
- Only abort the operation if the user requests it.
- Do not push as part of this skill unless explicitly asked.
- If unexpected changes appear, stop and ask the user how to proceed.

## Output Style

When finishing, report:

1. Which operation was in progress.
2. The files resolved and how each was decided.
3. Anything that required a user decision.
4. The validation performed.
5. The final git state (operation completed, or still in progress).
