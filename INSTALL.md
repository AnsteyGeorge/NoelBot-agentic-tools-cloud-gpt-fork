# Install

A one-time setup prompt. Paste it into any coding agent (or run the script yourself) to register every skill in this repo with each AI coding agent installed on this machine.

Skills are linked, not copied — so editing a skill here updates it everywhere at once. Re-run any time you add a new skill.

## Prompt

> Register every skill in this repository with all AI coding agents installed on my machine.
>
> 1. Treat each directory containing a `SKILL.md` as one skill, **except** the repo root. Read each `SKILL.md` frontmatter to get its `name:` (fall back to the directory name).
> 2. Detect which harnesses are present using the table below. A harness counts as present if its home directory exists **or** its binary is on `PATH`.
> 3. For each present harness, create its skills directory if needed, then **symlink** each skill directory into it as `<skills-dir>/<name>` pointing at the absolute path of the skill directory in this repo.
> 4. Be idempotent and safe: if a correct symlink already exists, skip it; if a wrong symlink exists, repoint it; if a real (non-symlink) file or directory already occupies that name, leave it alone and report it.
> 5. Print a summary: which harnesses were detected, which were skipped (not installed), and every skill that was linked, repointed, skipped, or conflicted.

## Harness reference

| Harness | Detect (dir or binary) | Skills directory |
|---|---|---|
| claude (Claude Code) | `~/.claude` or `claude` | `~/.claude/skills` |
| codex (Codex CLI) | `~/.codex` or `codex` | `~/.codex/skills` |
| cursor (Cursor CLI) | `~/.cursor` or `cursor` / `cursor-agent` | `~/.cursor/skills` |
| openclaw | `~/.openclaw` or `openclaw` | `~/.openclaw/skills` |
| hermes | `~/.hermes` or `hermes` | `~/.hermes/skills` |

All listed harnesses use the same convention: a personal skills directory containing one subdirectory per skill, each holding a `SKILL.md`. To support another agent, add a row with its home directory, binary, and skills path — the logic below is data-driven.

## Script (reliable path)

Run this from anywhere inside the repo. It does exactly what the prompt describes.

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_ROOT="$(cd "$(git -C "$(dirname "${BASH_SOURCE[0]:-$PWD}")" rev-parse --show-toplevel 2>/dev/null || echo "$PWD")" && pwd)"

# harness: "name|detect_dir|binary1,binary2|skills_dir"
HARNESSES=(
  "claude|$HOME/.claude|claude|$HOME/.claude/skills"
  "codex|$HOME/.codex|codex|$HOME/.codex/skills"
  "cursor|$HOME/.cursor|cursor,cursor-agent|$HOME/.cursor/skills"
  "openclaw|$HOME/.openclaw|openclaw|$HOME/.openclaw/skills"
  "hermes|$HOME/.hermes|hermes|$HOME/.hermes/skills"
)

# Collect skill directories (any dir with a SKILL.md), excluding repo root and _install.
mapfile -t SKILL_DIRS < <(
  find "$REPO_ROOT" -type f -name SKILL.md \
    -not -path "$REPO_ROOT/.git/*" \
    -not -path "$REPO_ROOT/_install/*" \
    -printf '%h\n' | sort -u
)
[ "${#SKILL_DIRS[@]}" -gt 0 ] || { echo "No skills found under $REPO_ROOT"; exit 0; }

skill_name() {
  local f="$1/SKILL.md" n
  n="$(awk -F: '/^name:[[:space:]]*/{sub(/^name:[[:space:]]*/,""); gsub(/[[:space:]]/,""); print; exit}' "$f")"
  [ -n "$n" ] && echo "$n" || basename "$1"
}

present() { # detect_dir + comma-separated binaries
  [ -d "$1" ] && return 0
  local IFS=','; for b in $2; do command -v "$b" >/dev/null 2>&1 && return 0; done
  return 1
}

for entry in "${HARNESSES[@]}"; do
  IFS='|' read -r name detect_dir bins skills_dir <<<"$entry"
  if ! present "$detect_dir" "$bins"; then
    echo "skip $name (not installed)"
    continue
  fi
  echo "== $name -> $skills_dir"
  mkdir -p "$skills_dir"
  for dir in "${SKILL_DIRS[@]}"; do
    src="$(cd "$dir" && pwd)"
    link="$skills_dir/$(skill_name "$dir")"
    if [ -L "$link" ]; then
      if [ "$(readlink -f "$link")" = "$src" ]; then echo "  ok      $(basename "$link")"; else ln -sfn "$src" "$link"; echo "  repoint $(basename "$link")"; fi
    elif [ -e "$link" ]; then
      echo "  CONFLICT $(basename "$link") (real path exists, left untouched)"
    else
      ln -s "$src" "$link"; echo "  link    $(basename "$link")"
    fi
  done
done
```
