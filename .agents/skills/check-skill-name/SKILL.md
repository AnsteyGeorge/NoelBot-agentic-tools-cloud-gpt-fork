---
name: check-skill-name
description: Check whether a proposed skill name collides with a reserved built-in command or bundled skill from Claude Code, Codex, Cursor, Hermes, or OpenClaw, or with a skill/agent already in this repo, and report a clear verdict plus non-conflicting alternatives. Conflicts are loaded from a local list (known-names.json) that an update mode refreshes from each tool's official docs. Use when naming a new skill or command, before creating its directory, or when the user asks whether a name is taken, safe, or reserved.
---

# Check Skill Name

## Core Contract

Use this skill to validate a proposed skill or slash-command name *before* it is created, so it never shadows — or gets shadowed by — a built-in command, bundled skill, or another tool's skill once installed across harnesses.

Names matter because global skills from this repo are symlinked into harness skill directories (for example `~/.claude/skills`, `~/.codex/skills`, `~/.cursor/skills`) and repo-local authoring skills are loaded from `.agents/skills` (with `.claude` pointing at `.agents`). A name that matches a harness built-in (e.g. `clear`, `review`, `model`) is invoked with `/name`, so a collision is shadowed or ambiguous. The job here is to catch that collision early.

The reserved-name list lives next to this file in `known-names.json`. It is the source of truth for the check; keep it current with the update mode below. This skill **reads and reports** — its only write is to `known-names.json` during an explicit update.

Keep `CLAUDE.md` / `AGENTS.md` in the repo authoritative; if they define naming conventions, apply them on top of this check.

## Required Inputs

1. **The proposed name** to check (one or several). If the user gave a `/slash` form or a directory path, reduce it to the bare name. If none was given, ask for it.
2. **Mode** — *check* (default) or *update* (refresh `known-names.json` from docs). Infer *update* when the user says the list is stale, asks to refresh sources, or a check result is suspected out of date.
3. For *check* mode, the **repo skills/agents** in scope — read from this repo so in-repo duplicates are caught too.

## Workflow

### Check mode (default)

1. **Normalize** the candidate: lowercase, trim, strip a leading `/`, and reduce any path to its basename. Note if it violates the naming rule (lowercase, hyphenated, no spaces/underscores) — that is its own kind of "bad name" worth flagging.
2. **Load reserved names** from `known-names.json` (the `names` map, grouped by source) and note the list's `updated` date.
3. **Load in-repo names**: collect the `name:` frontmatter of every `skills/**/SKILL.md`, `.agents/skills/**/SKILL.md`, and `agents/**/*.md` in this repo (see Implementation Notes for the command).
4. **Compare**:
   - **Exact match** against any reserved or in-repo name → **conflict** (hard block). Report which source(s) own it.
   - **Near match** (differs only by hyphen/underscore/pluralization, or is a known alias such as `bg`↔`background`, `tp`↔`teleport`, `proactive`↔`loop`) → **risky**; explain the ambiguity.
   - **No match** → **clear**.
5. **If conflicting or risky**, propose 3–5 alternative names that are free across all sources and the repo, fit the concept, and follow the naming rule. Prefer specific, concept-named alternatives over generic suffixes like `-v2`.
6. **Report** the verdict (see Output Style). If the list's `updated` date is old, say so and offer update mode.

### Update mode

Refresh `known-names.json` from the canonical docs so checks stay accurate.

1. Read the `sources` array in `known-names.json` for the list of `{id, label, url}` to refresh.
2. For each source, `WebFetch` its `url` and extract the bare command/skill names (the token after each `/`). Follow cross-host redirects the fetch reports.
3. Merge per source: union the freshly fetched names with the existing ones, drop obvious non-commands (frontmatter keys, prose), lowercase, and de-duplicate. Do not silently delete a name that the fetch missed — docs pages render inconsistently; only remove a name when you are confident it is gone, and say which.
4. Rewrite `known-names.json` with the merged `names`, a refreshed `updated` date (use today's date), and any source URL that changed.
5. Report what changed: added / removed / unchanged counts per source, and any source whose fetch failed (left as-is).

To add a new tool to track, append a `{id, label, kind, url}` entry to `sources` and a matching key under `names`, then run update mode.

## Implementation Notes

- Prefer one pass with `rg` over multi-process `find|xargs|awk` pipelines. Collect in-repo names from frontmatter with:
  ```bash
  rg -n --no-heading '^name:\s*' skills/**/SKILL.md .agents/skills/**/SKILL.md agents/**/*.md \
  | sed -E 's/.*name:\s*//' | tr -d '[:space:]' | sort -u
  ```
  This mirrors how `INSTALL.md` derives link names, so it catches the names that would collide on install while using fewer subprocesses.
- Read `known-names.json` with the Read tool and parse the `names` map. If `jq` is available, flatten with `jq -r '.names[][]' known-names.json | sort -u`.
- For faster checks, precompute normalized lookup sets in memory once per run:
  - exact names (for hard conflicts),
  - dehyphenated/deunderscored forms (for near-match risk checks),
  - alias map (`bg↔background`, etc.) from this file's alias list.
  Then resolve each candidate with constant-time set/map lookups.
- If checking many candidates, cache the in-repo name set for the run instead of rescanning files per candidate.
- Source URLs (also stored in the file): Claude Code → `code.claude.com/docs/en/commands`; Codex CLI → `developers.openai.com/codex/cli/slash-commands`; Cursor → `cursor.com/docs/cli/reference/slash-commands`; Hermes → `hermes-agent.nousresearch.com/docs/reference/slash-commands`; OpenClaw → `docs.openclaw.ai/tools/slash-commands`. Docs hosts redirect often — fetch what the redirect points to.
- Aliases share a command, so they count as conflicts: treat `bg/background`, `tp/teleport`, `proactive/loop` (Claude), `clean/stop`, `btw/side`, `approvals/permissions` (Codex), `reset/new`, `tasks/agents` (Hermes), and `think/thinking/t`, `verbose/v`, `reasoning/reason` (OpenClaw) as reserved too.
- The check is intentionally conservative: a name reserved by *any* one tool is a conflict, because skills install into all of them.

## Safety Rules

- Never create the skill directory or any skill files — this skill only validates a name. Creating the skill is a separate step the user takes after a clear verdict.
- The only file this skill may write is `known-names.json`, and only in update mode. Do not edit it during a check.
- In update mode, never blank out a source on a failed fetch — leave its existing names intact and report the failure.
- Do not invent names as "reserved" without a source; every reserved name traces to `known-names.json`.

## Output Style

For a **check**, report:

1. The normalized candidate and its verdict: **CLEAR**, **CONFLICT**, or **RISKY**.
2. For conflict/risky: every source that owns the colliding name (Claude Code / Codex / Cursor / Hermes / OpenClaw / this repo) and the exact matched name.
3. If not clear: 3–5 suggested alternatives that are free everywhere, with a one-line note on which concept each captures.
4. A footer line with the `known-names.json` `updated` date, and an offer to run update mode if it looks stale.

For an **update**, report per source: counts of added / removed / unchanged names, the new `updated` date, and any source that failed to refresh.
