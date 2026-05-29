---
name: pr-rebase
description: Rebases the current branch onto the latest origin/develop, resolves merge conflicts using branch purpose and the related PR description when available, then force-pushes with lease to origin. Use when the user wants to rebase a feature branch on develop, sync before merge, or update PR history after resolving conflicts.
disable-model-invocation: true
---

# PR rebase (develop)

Workflow-specific skill: lives under `.tmp/skills/pr-rebase/`. Not part of `.ai/skills` catalog.

## Core contract

1. Update local `develop`, rebase the **current** branch onto `origin/develop`.
2. When conflicts appear, decide using **this branch’s intent** plus the **open PR body/title** (if any). Prefer changes that preserve the feature’s goal and integrate upstream fixes correctly—do not blindly pick one side.
3. When the rebase completes successfully, push rewritten history with **`git push --force-with-lease`** to **`origin`** on the **current branch only**.

**Guards**

- Do not run on `develop` or `main`. If checked out there, stop or switch/create a feature branch per user instruction.
- Never force-push to `develop` or `main`.
- Prefer `--force-with-lease` over `--force` so a mismatched remote aborts instead of overwriting unknown work.

CLAUDE.md / AGENTS.md win if anything here conflicts.

---

## Intake (before rebase)

Record:

- Current branch: `git branch --show-current`
- Short intent: `git log origin/develop..HEAD --oneline` (or merge-base summary) so you know what the branch is trying to deliver.

**Related PR (optional but preferred)**

If `gh` is available and the repo tracks GitHub:

```sh
gh pr view --json title,body,number,url 2>/dev/null
```

If there is no PR or `gh` fails, use commit messages and user-stated goal only.

Treat PR title/body as the authority for *what this branch should accomplish* when choosing conflict resolutions.

---

## Steps

### 1. Fetch and rebase

```sh
git fetch origin develop
git rebase origin/develop
```

### 2. Conflicts

For each conflicted file:

1. Read conflict markers and both sides’ meaning in context (not only the hunk).
2. Align resolution with: **PR description/title** (if present) → **recent commits on this branch** → **minimal correct merge** with upstream (e.g. take upstream refactors, keep feature behavior).
3. Remove markers, ensure file is coherent (imports, types, formatting).
4. `git add` resolved paths.

When ambiguous after using PR + branch context, **ask the user** with a concrete question (file, options), do not guess.

Continue:

```sh
git rebase --continue
```

Repeat until rebase finishes or user aborts (`git rebase --abort` only if they ask).

### 3. Push

Only after a **successful** rebase (no in-progress rebase):

```sh
git push --force-with-lease origin HEAD
```

(or explicit branch name matching `git branch --show-current`.)

If `--force-with-lease` rejects: fetch, reconcile with the user (remote may have moved).

---

## Quick checklist

- [ ] Not on `develop` / `main`
- [ ] Fetched `origin/develop`, rebased onto it
- [ ] PR context loaded when possible (`gh pr view`)
- [ ] Conflicts resolved with intent + upstream consistency
- [ ] `git push --force-with-lease origin <branch>`
