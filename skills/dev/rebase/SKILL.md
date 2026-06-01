---
name: rebase
description: Rebase the current branch onto the latest base branch (main or develop), fetching first, automatically handing off to the conflicts skill when conflicts arise, then force-pushing with lease. Use when the user wants to rebase or update the current branch on top of its base.
---

# Rebase

## Core Contract

Use this skill when the user wants the current branch rebased onto the latest base (`main` or `develop`). This is the general, PR-agnostic local rebase; `pr-rebase` is the develop-only, PR-aware variant.

Conflict resolution is delegated to the `conflicts` skill. Do not duplicate that logic here.

Keep `CLAUDE.md` and `AGENTS.md` as mandatory policy sources. If this skill conflicts with them, follow those files.

## Required Inputs

Gather or infer:

1. The current branch (guard: it must not be the base itself).
2. The base branch to rebase onto (`main` or `develop`).
3. Whether to push afterward (default: yes, with `--force-with-lease`, after a clean rebase).

## Workflow

### 1. Intake and base detection

1. Run `git branch --show-current`. If on `main`, `develop`, or another protected base, stop and ask.
2. Determine the base branch: the repo default branch (`git remote show origin`, or `gh repo view --json defaultBranchRef`) or a `develop` convention if the repo uses one. If both `main` and `develop` exist and intent is unclear, ask which base.
3. Ensure a clean working tree. If there are uncommitted changes, stop and ask (offer to commit or stash first).
4. Summarize what the branch delivers: `git log origin/<base>..HEAD --oneline`.

### 2. Fetch

Run `git fetch origin <base>` (and the default remote) so the rebase targets up-to-date upstream.

### 3. Rebase

Run `git rebase origin/<base>`.

### 4. Handle conflicts via the conflicts skill

If the rebase stops with conflicts, hand off to the **`conflicts`** skill — invoke it via the Skill tool where the harness supports skill invocation, otherwise follow its workflow. It resolves each file by branch intent and runs `git rebase --continue`, looping until the rebase finishes. Return here once the rebase completes.

### 5. Push

1. After a clean, completed rebase, push the rewritten history with `git push --force-with-lease` on the current feature branch only.
2. If `--force-with-lease` is rejected, the remote moved: fetch and reconcile with the user. Never use plain `--force`.

### 6. Report

Summarize the result.

## Implementation Notes

- Useful commands: `git fetch`, `git rebase origin/<base>`, `git rebase --abort` (only if the user asks), `git push --force-with-lease`.
- Base detection: `git remote show origin`, or `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
- Conflicts: defer entirely to `skills/dev/conflicts`.

## Safety Rules

- Never run on, or force-push, `main`, `develop`, or any protected base.
- Never rebase with a dirty working tree; commit or stash first (ask the user).
- Use `--force-with-lease`, never plain `--force`.
- Do not resolve conflicts inline — delegate to `conflicts`.
- Only abort the rebase if the user asks.
- If `--force-with-lease` is rejected, stop and reconcile rather than overwriting.

## Output Style

When finishing, report:

1. The base branch rebased onto.
2. The commits replayed.
3. Whether conflicts occurred and that `conflicts` handled them.
4. The final state.
5. Whether and how the branch was pushed.
