<div align="center">

<br/>

<img alt="claude-code-hooks-cookbook" src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=800&size=36&duration=2400&pause=900&color=A78BFA&center=true&vCenter=true&width=900&height=80&lines=claude-code-hooks-cookbook"/>

**30+ production-ready Claude Code hooks. Copy. Paste. Profit.**
_Auto-format · security audit · license check · pre-commit guards · cost limits · and more._

<br/>

```bash
npx claude-hooks add format-on-write
```

<br/>

<p>
<img src="https://img.shields.io/badge/Claude%20Code-D97757?style=for-the-badge&logo=anthropic&logoColor=white"/>
<img src="https://img.shields.io/badge/Hooks-A78BFA?style=for-the-badge"/>
<img src="https://img.shields.io/badge/30%2B-FC60A8?style=for-the-badge"/>
<img src="https://img.shields.io/badge/MIT-000000?style=for-the-badge"/>
</p>

<p>
<img src="https://img.shields.io/github/stars/kasimmj/claude-code-hooks-cookbook?style=social"/>
<img src="https://img.shields.io/github/forks/kasimmj/claude-code-hooks-cookbook?style=social"/>
</p>

</div>

---

## 🪝 What are hooks?

Hooks are shell commands Claude Code runs automatically on events — file writes, tool calls, session start, session end. They're how you **enforce policy on Claude's actions** without trusting Claude to remember.

```json
// .claude/settings.json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "command": "./hooks/block-rm-rf.sh"
    }],
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "command": "./hooks/format-on-write.sh $CLAUDE_FILE"
    }]
  }
}
```

But writing good hooks takes time. **This repo has 30+ pre-built, tested ones.**

---

## ⚡ Quick Install

```bash
# Install a single hook
npx claude-hooks add format-on-write

# Install a recipe (curated set)
npx claude-hooks add-recipe safe-prod

# List all available
npx claude-hooks list

# Update all installed hooks to latest
npx claude-hooks update
```

The CLI edits your `.claude/settings.json` for you. No JSON wrangling.

---

## 📂 Catalog

### 🛡️ Safety Hooks
| Hook | Event | What it does |
|------|-------|--------------|
| `block-rm-rf` | PreToolUse(Bash) | Refuses dangerous deletion commands |
| `block-force-push` | PreToolUse(Bash) | No `git push --force` to main/master |
| `block-prod-credentials` | PreToolUse(Read) | Refuses to read `.env.prod`, secrets dirs |
| `confirm-destructive-sql` | PreToolUse(Bash) | Confirms before DROP/TRUNCATE in DB |
| `audit-shell-commands` | PreToolUse(Bash) | Logs every Bash command to audit.log |

### 🎨 Format & Quality
| Hook | Event | What it does |
|------|-------|--------------|
| `format-on-write` | PostToolUse(Write,Edit) | Auto-runs prettier/ruff/gofmt on changed files |
| `eslint-fix` | PostToolUse(Write,Edit) | Auto-applies fixable ESLint suggestions |
| `lint-on-stop` | Stop | Runs project linter at session end, shows results |
| `test-on-write` | PostToolUse(Write,Edit) | Runs tests matching the changed file |

### 📝 Documentation
| Hook | Event | What it does |
|------|-------|--------------|
| `update-readme-toc` | PostToolUse(Write) | Regenerates TOC if README changed |
| `enforce-jsdoc` | PostToolUse(Write,Edit) | Fails if new public APIs lack docs |

### 🚦 Pre-Commit
| Hook | Event | What it does |
|------|-------|--------------|
| `block-todo-fixme-prod` | PreToolUse(Bash:git commit) | Blocks commits with TODO/FIXME on main |
| `enforce-conventional-commit` | PreToolUse(Bash:git commit) | Refuses non-conventional messages |
| `block-large-files` | PreToolUse(Bash:git add) | Refuses files > 1MB |
| `secrets-scan` | PreToolUse(Bash:git commit) | Runs gitleaks/trufflehog before commit |

### 💰 Cost & Limits
| Hook | Event | What it does |
|------|-------|--------------|
| `cost-limit-session` | Stop | Tracks token cost, abort if > $X |
| `block-large-context` | UserPromptSubmit | Refuses prompts over N tokens |
| `model-selector` | UserPromptSubmit | Auto-downgrade trivial tasks to Haiku |

### 🌐 Workflow
| Hook | Event | What it does |
|------|-------|--------------|
| `notify-on-stop` | Stop | macOS/Slack notification when Claude finishes |
| `branch-name-enforce` | PreToolUse(Bash:git checkout) | Refuses non-conforming branch names |
| `auto-stash-on-switch` | PreToolUse(Bash:git checkout) | Stashes dirty work automatically |

### 🤖 AI Augment
| Hook | Event | What it does |
|------|-------|--------------|
| `inject-conventions` | UserPromptSubmit | Prepends project conventions to every prompt |
| `inject-git-context` | UserPromptSubmit | Adds current git status/branch to every prompt |
| `ai-format-commit` | PostToolUse(Bash:git commit) | Re-writes weak commit messages |

---

## 🍱 Recipes

Pre-curated bundles for common stacks:

```bash
npx claude-hooks add-recipe safe-prod         # All safety hooks
npx claude-hooks add-recipe js-fullstack      # Format + ESLint + tests for JS
npx claude-hooks add-recipe python-data       # Ruff + pytest + mypy
npx claude-hooks add-recipe rust-strict       # rustfmt + clippy + cargo check
npx claude-hooks add-recipe team-standard     # The hooks every team should have
```

---

## 🛠️ Anatomy of a Hook

Every hook in this repo is a single shell script + a JSON snippet to add to `settings.json`.

Example: `format-on-write`

**`hooks/format-on-write.sh`:**
```bash
#!/usr/bin/env bash
# format-on-write — auto-format a file based on its extension
set -euo pipefail

file="${CLAUDE_FILE:-}"
[[ -z "$file" ]] && exit 0

case "$file" in
  *.ts|*.tsx|*.js|*.jsx|*.json|*.md|*.css|*.html)
    command -v prettier >/dev/null && prettier --write "$file" 2>/dev/null || true
    ;;
  *.py)
    command -v ruff >/dev/null && ruff format "$file" 2>/dev/null || true
    ;;
  *.go)
    command -v gofmt >/dev/null && gofmt -w "$file" 2>/dev/null || true
    ;;
  *.rs)
    command -v rustfmt >/dev/null && rustfmt "$file" 2>/dev/null || true
    ;;
esac
```

**JSON snippet (auto-added to settings.json):**
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "command": "$CLAUDE_PROJECT_DIR/.claude/hooks/format-on-write.sh"
    }]
  }
}
```

---

## 🤝 Contributing

PRs welcome. A hook gets accepted if it:

1. **Solves a concrete, repeatable problem**
2. **Is tested across macOS/Linux** (shell-only, no platform-specific deps unless flagged)
3. **Has a clear name** (verb-object format: `format-on-write`, `block-rm-rf`)
4. **Includes a `description.md`** explaining when to use it and when NOT
5. **Is idempotent** — running it twice doesn't break things

---

## 📜 License

MIT.

---

<div align="center">

**Star ⭐ to follow the hook collection.**

</div>
