---
name: codex-cli
description: Call Codex CLI for independent code review, second-opinion reasoning, sandboxed execution, and live web search.
---

# Codex CLI

Codex CLI runs headless as an AI colleague. For rare flags, run `codex exec --help` at runtime.

## Flag Placement (Critical)

Flag placement is strict — wrong level causes `unexpected argument`:
- **Before `exec`** (top-level): `-a never`, `--search`, `-i`
- **After `exec`** (exec-level): `-s`, `--skip-git-repo-check`, and all other exec options
- **After `review`/`resume`** (subcommand): `--uncommitted`, `--base`, `--last`, etc.

## Standard Call

```bash
PROMPT='your prompt here'
codex -a never exec --skip-git-repo-check -s <sandbox> "$PROMPT" 2>/dev/null
```

| Element | Why |
|---|---|
| `-a never` | **Before `exec`.** Prevents interactive blocking — required for agent-to-agent. |
| `--skip-git-repo-check` | Always include — avoids trusted-directory errors. |
| `-s <sandbox>` | Always explicit. Default to `read-only`; use `workspace-write` only when edits needed. |
| `2>/dev/null` | Suppresses noise. Omit when debugging. |
| Quoted `$PROMPT` | Never interpolate raw user text. Use variable or stdin (`echo '...' \| codex ... -`). |
| `timeout: 300000` | 5-min timeout on Bash tool calls. |

**Common exec-level options** (place after `exec`):
| Flag | Purpose |
|---|---|
| `-c model_reasoning_effort="<level>"` | Claude decides per task: `low` (trivial), `medium` (standard), `high` (thorough), `xhigh` (complex). Syntax not in `--help`. |
| `-m <model>` | Omit for default. Claude may escalate for complex tasks; user may override. |
| `--json` | JSONL event stream on stdout for programmatic parsing. |
| `-o <file>` | Write final message to file. Combine with `--json` for both. |
| `-C <dir>` | Set working directory. |

## Web Search

`--search` is **top-level** (before `exec`):
```bash
codex -a never --search exec --skip-git-repo-check -s read-only "$PROMPT" 2>/dev/null
```

## Review

```bash
codex -a never exec --skip-git-repo-check -s read-only review --uncommitted "$PROMPT" 2>/dev/null
```

## Resume

```bash
echo 'follow-up' | codex -a never exec --skip-git-repo-check resume --last 2>/dev/null
```
Exec-level flags go between `exec` and `resume`; `--last` goes after `resume`. Default to fresh sessions; only resume for multi-step workflows.

## Troubleshooting

- **Hangs** → missing `-a never`. **`unexpected argument`** → flag at wrong level.
- Treat Codex as a colleague, not an authority — verify surprising claims independently.
