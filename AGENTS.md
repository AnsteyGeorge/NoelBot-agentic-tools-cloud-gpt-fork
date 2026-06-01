# AGENTS.md

Guidance for AI agents working **on** this repository.

## What this repo is

A collection of reusable **agentic tools**, shared across machines and projects.
Two kinds of tool live here today:

- **Skills** — each lives in its own directory containing a single `SKILL.md`,
  grouped into category directories under `skills/`. Skills follow the
  agentskills.io standard and are portable across harnesses.
- **Agent profiles** — Claude Code subagents, single `.md` files under
  `agents/`. These are Claude-Code-specific.

Tools are **installed by symlink, not copy** (see `INSTALL.md`), so editing a
tool here updates it in every harness at once. Skills install into every
detected harness; agent profiles install only into harnesses that support them
(Claude today). `README.md` is the human-facing catalog. `INSTALL.md` is the
one-time setup that links every tool into each installed harness.

Special scope rule: repo-local skill-authoring helpers live under
`.agents/skills/` in this repo. `.claude` is a symlink to `.agents`, so Claude
sees the same files at `.claude/skills/` without duplication.

Scope expectations by location:

- Skills under `skills/` are global artifacts. Write them to be repository- and
  content-agnostic so they transfer cleanly across projects.
- Skills under `.agents/skills/` are repo-local helpers. They may be
  opinionated to this repository and should follow this `AGENTS.md` as the
  primary convention source.

## Repository layout

```
skills/<category>/<skill-name>/SKILL.md   # one globally-installed skill
.agents/skills/<skill-name>/SKILL.md      # one repo-local skill (authoring helpers)
agents/[<group>/]<agent-name>.md          # one agent profile
README.md                                 # catalog of all tools
INSTALL.md                                # symlink-install prompt + script
AGENTS.md                                 # this file
```

Tools are keyed by their `name:` frontmatter, not their path — `skills/`
category folders, `.agents/skills/`, and any `agents/` grouping folders are
purely organizational and invisible to the harness.

- `skills/` must **not** contain a `SKILL.md` at its own root; the installer
  treats every directory with a `SKILL.md` as one skill.
- Keep role separation: skill `SKILL.md` files live under `skills/` (global) or
  `.agents/skills/` (repo-local authoring helpers); agent `.md` profiles live
  under `agents/`.

## SKILL.md conventions

Every skill follows the same shape. Match it exactly when adding or editing one.

**Frontmatter** — YAML with four keys:

```yaml
name: skill-name
description: One paragraph describing what the skill does, the concrete steps it covers, and a "Use when ..." trigger sentence.
user-invocable: true
disable-model-invocation: false
```

- `name` is lowercase, hyphenated, and **must** match the skill's directory
  name. The installer symlinks `<skills-dir>/<name>`, so a mismatch creates a
  confusing link.
- `description` is third person, names the major steps the skill performs, and
  ends with a sentence starting with "Use when ...".
- `user-invocable` controls whether users can call the skill directly. Default
  is `true`.
- `disable-model-invocation` controls whether the model can auto-invoke the
  skill from context. Default is `false`.

**Body** — these sections, in order:

1. `# Title`
2. `## Core Contract` — when to use the skill, default scope, external tools.
   `CLAUDE.md` / `AGENTS.md` in the target repo are the source of truth and
   override the skill on conflict.
3. `## Required Inputs` — what to gather or infer before acting.
4. `## Workflow` — numbered steps, with explicit stop-and-ask gates.
5. `## <Platform> Implementation Notes` — concrete commands.
6. `## Safety Rules` — hard constraints ("Never ...").
7. `## Output Style` — what to report when done.

## Agent profile conventions

An agent profile is a Claude Code subagent: a single `.md` file whose body is the
subagent's system prompt.

**Location** — `agents/<name>.md`, optionally grouped in subfolders
(`agents/review/code-reviewer.md`). The path is cosmetic; Claude identifies the
agent by its `name:` frontmatter.

**Frontmatter** — YAML:

```yaml
name: agent-name          # required; lowercase-hyphenated; unique repo-wide
description: Third-person summary of what the agent does, ending with a "Delegate when ..." trigger so Claude knows when to route to it.   # required
tools: Read, Grep, Glob, Bash   # optional; least-privilege allowlist
model: inherit                  # optional; inherit | sonnet | haiku | opus
```

- `name` must be unique across the whole repo (it becomes the install link
  `<agents-dir>/<name>.md`) and should match the filename for human sanity.
- Restrict `tools` to the least privilege the role needs; omit to inherit all.

**Body** — the markdown body **is the system prompt**. Write focused, imperative
instructions for one role. It must be self-contained: a subagent receives only
this body plus the environment, not the full Claude Code system prompt. Mirror
the skills' "stop and ask the user" guidance and the
"CLAUDE.md/AGENTS.md in the target repo override this" clause.

## Adding a new tool

- **New global skill:** add `skills/<category>/<name>/SKILL.md` following the
  conventions above; keep behavior repo/content agnostic; mirror an existing skill such as
  `skills/pull-requests/pr-ci/SKILL.md`.
- **New repo-local authoring skill:** add `.agents/skills/<name>/SKILL.md`;
  mirror `.agents/skills/make-skill/SKILL.md` and make it explicitly
  repository-opinionated where useful.
- **New agent:** add `agents/<name>.md`; mirror `agents/code-reviewer.md`.

No installer change is needed for a new global skill or agent of an
already-supported type — the script discovers it. Re-run `INSTALL.md` to link
it. Renaming a tool means renaming the file/directory **and** its `name:`
together, then re-running install (it repoints) and removing any stale links.

## Commit granularity policy

For this repository, treat edits to individual skills as separate units of
change by default.

- Default commit shape: one skill per commit (`skills/.../<name>/SKILL.md` or
  `.agents/skills/<name>/SKILL.md`).
- Bundle multiple skills into one commit only when they are lock-step coupled:
  they reference each other directly, share one contract/schema change, or must
  ship together to avoid a broken intermediate state.
- When bundling is chosen, the commit message/body should state why lock-step
  coupling requires a single commit.

## Things to avoid

- Don't add a `SKILL.md` at the `skills/` root, or place agent `.md` files under
  `skills/`/`.agents/skills/` (or vice-versa) — it breaks type-scoped discovery.
- Don't let a tool's `name:` drift from its file/directory name.
- Don't copy a tool into a harness directory; the install flow symlinks.
- Don't commit harness-specific or absolute-path artifacts into the repo.
- Don't create new top-level docs or markdown files unless asked.
- Don't break the data-driven install script in `INSTALL.md` — to support a new
  harness or artifact type, extend the harness table (add a row or a `type:dir`
  pair) rather than special-casing logic.
