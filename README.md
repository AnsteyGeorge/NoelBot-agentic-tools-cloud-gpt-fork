# Skills

Agent skills I use to do real engineering work. Each skill is a small, self-contained `SKILL.md` that teaches a coding agent how to run one workflow reliably, end-to-end.

They're designed to be composable and easy to adapt. Read one, hack on it, make it your own.

## Pull Requests

Skills for keeping branches and PRs healthy on GitHub. They default to `gh`, discover the PR from the current branch, and treat `CLAUDE.md` / `AGENTS.md` as the source of truth.

- **[pr-ci](pull-requests/pr-ci/SKILL.md)** — Investigate and fix failed PR CI jobs end-to-end: discover the PR, read failed-job logs, deduplicate failures by root cause, fix the code, validate, commit, and push.
- **[pr-comments](pull-requests/pr-comments/SKILL.md)** — Address unresolved PR review comments end-to-end: triage each thread, make worthwhile changes, validate, commit, push, then reply to or resolve every thread.
- **[pr-description](pull-requests/pr-description/SKILL.md)** — Refresh a PR description so it matches the current changeset: analyze drift against the base branch, rewrite sections concisely, maintain checklists, and update the body via `gh`.
- **[pr-rebase](pull-requests/pr-rebase/SKILL.md)** — Rebase the current branch onto the latest `origin/develop`, resolve conflicts using branch intent and the PR description, then force-push with lease.
- **[pr-restack](pull-requests/pr-restack/SKILL.md)** — Re-align a stack of dependent branches/PRs after upstream branches drift, rebase, force-push, or merge — re-pointing each downstream branch at its correct base while preserving its own commits.
