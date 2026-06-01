# INSTALL

A one-time setup prompt to register every tool in this repo with each AI coding
agent installed on the machine.

Tools are **linked, not copied** — editing a `SKILL.md` or an agent profile here
updates it in every harness at once. Re-run whenever you add a new tool or set up
a new machine; the script is idempotent.

This repo ships two kinds of tool:

- **Skills** (`skills/<category>/<name>/SKILL.md`) — portable across harnesses
  (agentskills.io standard). Installed into every detected harness.
- **Agent profiles** (`agents/[<group>/]<name>.md`) — Claude Code subagents.
  Installed only into harnesses that support them (Claude today).

## Prompt

> Register every tool in this repository with all AI coding agents installed on
> this machine.
>
> 1. Treat each directory under `skills/` containing a `SKILL.md` as one skill,
>    and each `*.md` file under `agents/` as one agent profile. The link name is
>    the tool's `name:` frontmatter (fall back to the directory/file basename).
> 2. Detect which harnesses are present — by home directory or by binary on
>    `PATH` — and which artifact types each one supports. This is data-driven:
>    each harness declares a map of `type:destination` pairs.
> 3. For each present harness, symlink every supported tool into the matching
>    destination: skills as directory symlinks, agent profiles as file symlinks
>    named `<name>.md`.
> 4. Be idempotent: skip links that are already correct, repoint links that
>    point elsewhere, and never clobber a real (non-symlink) file — report it as
>    a conflict and leave it untouched.
>
> Run the script below, or do exactly what it describes.

## Script

Run this from anywhere inside the repo.

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_ROOT="$(cd "$(git -C "$(dirname "${BASH_SOURCE[0]:-$PWD}")" rev-parse --show-toplevel 2>/dev/null || echo "$PWD")" && pwd)"

# Each harness declares: name | detect_dir | binaries | type:dir;type:dir;...
# A harness installs only the artifact types it lists. Add a "type:dir" pair to
# teach a harness about a new type; add a row to support a new harness.
HARNESSES=(
  "claude|$HOME/.claude|claude|skills:$HOME/.claude/skills;agents:$HOME/.claude/agents"
  "codex|$HOME/.codex|codex|skills:$HOME/.codex/skills"
  "cursor|$HOME/.cursor|cursor,cursor-agent|skills:$HOME/.cursor/skills"
  "openclaw|$HOME/.openclaw|openclaw|skills:$HOME/.openclaw/skills"
  "hermes|$HOME/.hermes|hermes|skills:$HOME/.hermes/skills"
)

# Extract the `name:` value from a file's YAML frontmatter.
fm_name() {
  awk -F: '/^name:[[:space:]]*/{sub(/^name:[[:space:]]*/,"");gsub(/[[:space:]]/,"");print;exit}' "$1"
}

# Skills: directories with a SKILL.md under skills/. Emits "linkname<TAB>srcdir".
collect_skills() {
  [ -d "$REPO_ROOT/skills" ] || return 0
  find "$REPO_ROOT/skills" -type f -name SKILL.md -not -path '*/.git/*' -printf '%h\n' | sort -u |
  while read -r d; do
    n="$(fm_name "$d/SKILL.md")"; [ -n "$n" ] || n="$(basename "$d")"
    printf '%s\t%s\n' "$n" "$(cd "$d" && pwd)"
  done
}

# Agents: *.md files under agents/. Emits "linkname.md<TAB>srcfile".
collect_agents() {
  [ -d "$REPO_ROOT/agents" ] || return 0
  find "$REPO_ROOT/agents" -type f -name '*.md' -not -path '*/.git/*' |
  while read -r f; do
    n="$(fm_name "$f")"; [ -n "$n" ] || n="$(basename "${f%.md}")"
    printf '%s.md\t%s\n' "$n" "$(cd "$(dirname "$f")" && pwd)/$(basename "$f")"
  done
}

# Present if the home dir exists or any listed binary is on PATH.
present() {
  [ -d "$1" ] && return 0
  local IFS=','; for b in $2; do command -v "$b" >/dev/null 2>&1 && return 0; done
  return 1
}

# Idempotent linker shared by all types (file or directory source).
link_one() {
  local link="$1" src="$2"
  if [ -L "$link" ]; then
    if [ "$(readlink -f "$link")" = "$src" ]; then echo "  ok       $(basename "$link")"
    else ln -sfn "$src" "$link"; echo "  repoint  $(basename "$link")"; fi
  elif [ -e "$link" ]; then
    echo "  CONFLICT $(basename "$link") (real path exists, left untouched)"
  else
    ln -s "$src" "$link"; echo "  link     $(basename "$link")"
  fi
}

for entry in "${HARNESSES[@]}"; do
  IFS='|' read -r name detect_dir bins typemap <<<"$entry"
  if ! present "$detect_dir" "$bins"; then echo "skip $name (not installed)"; continue; fi
  echo "== $name"
  IFS=';' read -ra pairs <<<"$typemap"
  for pair in "${pairs[@]}"; do
    type="${pair%%:*}"; dir="${pair#*:}"
    case "$type" in
      skills) collector=collect_skills ;;
      agents) collector=collect_agents ;;
      *) echo "  [$type] unknown type, skipping"; continue ;;
    esac
    echo "  [$type] -> $dir"
    mkdir -p "$dir"
    while IFS=$'\t' read -r linkname src; do
      [ -n "$linkname" ] || continue
      link_one "$dir/$linkname" "$src"
    done < <($collector)
  done
done
```
