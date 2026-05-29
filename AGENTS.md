# AGENTS.md

Guidance for AI agents working **on** this repository. This repo is a collection of agent skills — small, self-contained workflows that teach a coding agent how to run one task reliably, end-to-end. You are editing the skills themselves, not using them.

## What this repo is

- Each skill lives in its own directory containing a single `SKILL.md`.
- Skills are grouped into category directories (for example, `pull-requests/`).
- Skills are **installed by symlink**, not copy (see `INSTALL.md`). Editing a `SKILL.md` here updates it in every harness at once.
- `README.md` is the human-facing catalog. `INSTALL.md` is the one-time setup that links every skill into each installed agent harness.

## Repository layout

```
<category>/<skill-name>/SKILL.md   # one skill
README.md                          # catalog of all skills
INSTALL.md                         # symlink-install prompt + script
AGENTS.md                          # this file
```

The repo root must **not** contain a `SKILL.md`; the installer treats every directory with a `SKILL.md` (except the root) as one skill.

## SKILL.md conventions

Every skill follows the same shape. Match it exactly when adding or editing one.

### Frontmatter

YAML frontmatter with two keys:

```yaml
---
name: skill-name
description: One paragraph describing what the skill does, the concrete steps it covers, and a "Use when ..." trigger sentence.
---
```

- `name` is lowercase, hyphenated, and **must** match the skill's directory name. The installer symlinks the skill as `<skills-dir>/<name>`, so a mismatch creates a confusing link.
- `description` is written in the third person, names the major steps the skill performs, and ends with a sentence starting with "Use when ..." so a harness can decide when to invoke it.

### Body structure

After the frontmatter, use a `# Title` heading, then these sections (omit ones that genuinely don't apply, but keep the order):

1. `## Core Contract` — when to use the skill, the default scope (usually end-to-end), and which external tools it defaults to. State that `CLAUDE.md` / `AGENTS.md` in the target repo are the source of truth and override the skill on conflict.
2. `## Required Inputs` — what to gather or infer before acting.
3. `## Workflow` — numbered `###` steps, each with concrete, ordered actions. This is the heart of the skill.
4. Implementation notes (for example, `## GitHub Implementation Notes`) — concrete commands and recommended sequences.
5. `## Safety Rules` — a "Never ..." list of hard constraints.
6. `## Output Style` — what to report when finishing.

### Writing style

- Be imperative and specific. Prefer numbered steps and short directive sentences over prose.
- Tell the agent when to **stop and ask the user** (ambiguity, multiple matches, product/architecture decisions, risky changes).
- Make skills end-to-end by default but easy to scope down on request.
- Keep skills self-contained: a skill should not depend on another skill being present.
- Don't hardcode one harness's assumptions; skills should work across Claude Code, Codex, Cursor, and the other harnesses in `INSTALL.md`.

## Adding a new skill

1. Create `<category>/<skill-name>/SKILL.md` (create the category directory if needed). Reuse an existing category when it fits.
2. Write the frontmatter and body following the conventions above. Read a sibling skill (for example, `pull-requests/pr-ci/SKILL.md`) and mirror its structure.
3. Add a bullet to the matching section of `README.md` linking the new skill and summarizing it in one line.
4. No installer change is needed — `INSTALL.md`'s script discovers any directory with a `SKILL.md` automatically. Re-running the install prompt links the new skill everywhere.

## Editing or renaming a skill

- When editing a skill, preserve its section structure and tone.
- Renaming a skill means renaming **both** the directory and the `name:` frontmatter together, then updating `README.md`. Existing symlinks point at the old name, so the install step must be re-run and stale links cleaned up.
- Keep `README.md` in sync whenever a skill's purpose, name, or path changes.

## Things to avoid

- Don't add a `SKILL.md` at the repo root.
- Don't let `name:` drift from the directory name.
- Don't copy a skill into a harness directory; the install flow symlinks. Don't commit harness-specific or absolute-path artifacts into the repo.
- Don't create new top-level docs or markdown files unless asked.
- Don't break the data-driven install script in `INSTALL.md` — extend the harness table rather than special-casing logic.
