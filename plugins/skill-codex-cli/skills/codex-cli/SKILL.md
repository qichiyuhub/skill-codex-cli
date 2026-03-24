---
name: codex-cli
description: Call Codex CLI as your AI colleague for independent code review, second-opinion reasoning, sandboxed execution, live web search, structured output, and cross-validation when you need a fresh perspective.
---

# Codex CLI — AI Colleague & Wingman

Codex CLI is not an external service you reluctantly call — it is your colleague. Proactively leverage its strengths to compensate for your limitations.

## When to Call Codex (Decision Framework)

Actively consider calling Codex whenever you encounter these situations:

| Your Limitation | Codex's Strength | Example |
|---|---|---|
| Self-review bias | Independent code reviewer (`codex review`) | You wrote the code and need an unbiased second opinion |
| Reasoning uncertainty | Cross-validation with different model family | You are not confident in your conclusion — get a second opinion from OpenAI reasoning models |
| Knowledge cutoff | Live web search (`--search`) | User asks about events/releases/API changes after your cutoff date |
| No sandboxed execution | Three-tier sandbox (`read-only`, `workspace-write`, `danger-full-access`) | Need to run untrusted code or test in isolation |
| Image analysis second opinion | Multi-modal via `-i` flag (PNG, JPEG) | Need a second perspective on a screenshot, error UI, or design mockup |
| Stuck in logic loop | Fresh-eye review from different model | You have failed to fix a bug after 2+ attempts — call Codex for independent analysis |
| Need structured output | `--output-schema` + `--json` | Need machine-parseable results (JSON risk reports, TODO lists, change summaries) |
| Need resumable side task | Session resume (`codex exec resume --last`) | Long-running analysis that may need continuation |

**You do NOT need to wait for the user to say "use Codex".** If the situation matches the table above, call Codex proactively and tell the user why.

## Codex's Full Capability Set

### Core Tools
Codex operates by executing shell commands within a sandboxed environment. It can read files, write files, run commands, view diffs, and plan changes — all scoped by the sandbox mode you choose.

### Dedicated Code Review
- **`codex review`** / **`codex exec review`** — Specialized reviewer that reads your selected diff and outputs prioritized, actionable findings. **Does NOT modify the working tree.**
- Supports `--uncommitted` (staged + unstaged + untracked), `--base <branch>` (branch diff), `--commit <sha>` (single commit).

### Extended Capabilities
- **Multi-modal input:** Images (PNG, JPEG) via `-i/--image` flag. Useful for screenshots, error UIs, design mockups, architecture diagrams.
- **Live web search:** Built-in `web_search` tool. Default is **cached**; pass `--search` as a **top-level flag** (before `exec`) for **live** real-time results.
- **MCP client:** Can connect to external MCP servers (stdio and streamable HTTP, with bearer token/OAuth support).
- **MCP server:** Can itself run as `codex mcp-server` (stdio), exposing its capabilities to external callers.
- **Session management:** `codex exec resume --last` to continue a previous task. `--ephemeral` to disable session persistence.
- **Subagents:** Can spawn parallel subagents for complex task decomposition.
- **Structured output:** `--output-schema` forces final output to conform to a JSON Schema.

### Tool Management
```bash
codex mcp list
codex mcp add <name> <url>
codex mcp remove <name>
codex mcp login <name>
codex mcp logout <name>
```

## Command Structure

### Flag Placement Rules (Critical)

Codex CLI has a strict flag hierarchy. Placing flags at the wrong level causes parse errors:

```
codex [TOP-LEVEL FLAGS] exec [EXEC FLAGS] [review|resume] [SUBCOMMAND FLAGS] "<PROMPT>"
```

| Flag Level | Flags | Placement |
|---|---|---|
| **Top-level** (before `exec`) | `-a`/`--ask-for-approval`, `--search`, `-i`/`--image` | `codex -a never --search exec ...` |
| **Exec-level** (after `exec`) | `-s`/`--sandbox`, `--json`, `-o`, `--output-schema`, `--ephemeral`, `-C`, `--add-dir`, `--skip-git-repo-check`, `--full-auto`, `-m`, `-c` | `codex ... exec -s read-only --json ...` |
| **Subcommand** (after `review`/`resume`) | `--uncommitted`, `--base`, `--commit`, `--last` | `codex ... exec ... review --base main ...` |

### Primary: `codex exec` (Headless Worker)

```bash
codex -a never exec -s <sandbox> [options] "<PROMPT>"
```

This is the default way to call Codex from Claude Code. It runs non-interactively and returns results.

### Secondary: `codex review` (Dedicated Reviewer)

```bash
codex review --uncommitted "<REVIEW_INSTRUCTIONS>"
codex -a never exec -s read-only review --base main --json -o review.txt "<REVIEW_INSTRUCTIONS>"
```

Use `codex review` for quick human-readable output. Use `codex exec review` when you need `--json`, `--ephemeral`, or `-o`.

### Recommended Parameters for Plugin Calls
- **`-a never`** — Place before `exec`. Strongly recommended for agent-to-agent calls to prevent interactive blocking.
- **`-s <mode>`** — Place after `exec`. Strongly recommended — always specify explicitly rather than relying on user's local config.

### Key Options
| Flag | Level | Purpose |
|---|---|---|
| `-a, --ask-for-approval` | Top-level | `never` (headless, recommended), `untrusted` (only trusted cmds auto-approved), `on-request` (model decides) |
| `--search` | Top-level | Enable **live** web search (default is cached). |
| `-i, --image` | Top-level | Attach image file(s) to the prompt. |
| `-m, --model` | Exec | Model to use. Do not hardcode version IDs — use model names directly. |
| `-s, --sandbox` | Exec | `read-only` (analysis), `workspace-write` (edit files), `danger-full-access` (unrestricted, external sandbox only) |
| `--full-auto` | Exec | Shorthand for `-a on-request -s workspace-write`. **Not truly unattended** — may still prompt. Prefer explicit `-a never`. |
| `--json` | Exec | JSONL event stream on stdout. Use for programmatic consumption. |
| `-o, --output-last-message` | Exec | Write final message to file. |
| `--output-schema` | Exec | Force structured output conforming to a JSON Schema file. |
| `--ephemeral` | Exec | Disable session persistence. Cannot `resume` after this. |
| `-C, --cd` | Exec | Set working directory. |
| `--add-dir` | Exec | Additional writable directories beyond the primary workspace. |
| `--skip-git-repo-check` | Exec | Allow running outside a Git repository. |
| `-c, --config` | Exec | Override config values (e.g., `-c model="o3"`, `-c model_reasoning_effort="high"`). |

## Sandbox Mode Guide

Choose the minimum necessary sandbox level:

| Mode | When to Use | What's Allowed |
|---|---|---|
| `read-only` | **Recommended default.** Code analysis, architecture review, search, planning. | Read files, run read-only commands. No writes. |
| `workspace-write` | Bug fixes, refactoring, test writing. Most Claude-to-Codex write tasks. | Read + write within workspace and `--add-dir` directories. |
| `danger-full-access` | **Only in externally sandboxed environments** (VM, container, CI runner). Never as plugin default. | Full filesystem and network access. |

**Important:**
- `--full-auto` is just `workspace-write` + `on-request`. For agent-to-agent calls, prefer `codex -a never exec -s <mode>`.
- Need multi-directory writes? Use `--add-dir`, don't escalate to `danger-full-access`.

## Output Format Guide

| Method | When to Use |
|---|---|
| (default) | Human-readable final message on stdout. Simple queries, explanations. |
| `--json` | JSONL event stream. Programmatic consumption, CI/CD pipelines, structured processing. |
| `-o <file>` | Write final message to a file. Combine with `--json` for both streaming events and captured output. |
| `--output-schema <file>` | Force structured JSON output matching a schema. Risk reports, change summaries, TODO extraction. |

**Rule:** If Claude needs to programmatically parse Codex's output, use `--json`. Do NOT parse terminal text.

## Delegation Modes

Choose the mode based on what you need Codex to DO:

| Mode | Command Pattern | When to Use | Delegation Prefix |
|---|---|---|---|
| Read-only analysis | `codex -a never exec -s read-only` | **Recommended default.** Code review, search, architecture analysis, planning. | *"You are a secondary agent of the primary AI assistant. Do not modify any files."* |
| Workspace editing | `codex -a never exec -s workspace-write` | Bug fixes, refactoring, test writing within the repo. | *"You are a secondary agent of the primary AI assistant. Execute the task within the workspace."* |
| Full autonomy | `codex -a never exec -s danger-full-access` | **Only in externally sandboxed environments.** | *"You are a secondary agent of the primary AI assistant. Execute the task fully and autonomously."* |

**Important:** When Codex operates in `workspace-write` or higher, always review its changes before presenting to the user.

## Collaboration Patterns

### Independent Code Review
When you want Codex to review code you wrote or changes in the repo:
```bash
codex review --uncommitted "You are a secondary agent of the primary AI assistant. Review for: behavioral regressions, security issues, missing tests, API compatibility. Be specific and actionable."
```

For automation-friendly structured review:
```bash
codex -a never exec -s read-only review --base main --json -o review.txt "You are a secondary agent of the primary AI assistant. Focus on API compatibility and test coverage gaps."
```

### Cross-Validation
When you want Codex to independently verify your analysis:
```bash
PROMPT='You are a secondary agent of the primary AI assistant. Do not modify any files.

The primary agent analyzed the following problem and reached a conclusion. Please independently analyze and either confirm or challenge:

**Problem:** <describe the problem>
**Primary agent conclusion:** <your conclusion>
**Key files to read:** <file paths>

Provide your independent assessment. If you disagree, explain why with specific evidence.'
codex -a never exec -s read-only "$PROMPT" 2>/dev/null
```

### Live Web Search
When you need current information beyond your knowledge cutoff:
```bash
codex -a never --search exec -s read-only "You are a secondary agent of the primary AI assistant. Search for: <specific query>. Summarize the top results with source URLs." 2>/dev/null
```

### Structured Output Pipeline
When you need machine-parseable results:
```bash
codex -a never exec -s read-only --output-schema ./risk-schema.json -o ./risk-report.json "Analyze this codebase for security risks. Output as structured JSON per the schema." 2>/dev/null
```

### Peer Discussion (When You Disagree with Codex)
When Codex returns a result you believe is incorrect, resume the session and identify yourself:
```bash
echo "This is Claude (the primary agent) following up. I disagree with your conclusion about <X> because <evidence>. Please reconsider and provide your updated analysis." | codex -a never exec -s read-only resume --last - 2>/dev/null
```
Frame disagreements as discussions, not corrections — either AI could be wrong. Let the user decide if there's genuine ambiguity.

### Two-Phase Pipeline (Analyze then Fix)
```bash
codex -a never exec -s read-only "Find all race conditions in the codebase" 2>/dev/null
codex -a never exec -s workspace-write resume --last "Fix the issues you just found" 2>/dev/null
```

## Execution Rules

1. **Always use `-a never` before `exec`** for agent-to-agent calls. This prevents interactive approval prompts that would block Claude Code.
2. **Always specify `-s` after `exec`** explicitly. Do not rely on user's local default config.
3. **Suppress stderr noise:** Consider appending `2>/dev/null` to `codex exec` commands when you don't need diagnostic output. This suppresses progress indicators and reasoning traces that can bloat Claude Code's context. **Caution:** `2>/dev/null` also hides real errors (auth failures, parse errors). Omit it when debugging, when a command fails, or when the user explicitly requests full output.
4. **Reasoning effort:** Use `-c model_reasoning_effort="<level>"` to control depth. Values: `xhigh` (complex analysis), `high` (thorough), `medium` (standard), `low` (quick), `minimal` (lightest). Actual support depends on the model. This is an exec-level flag.
5. **Prompt Safety:** NEVER interpolate user input directly into the shell command string. Use a shell variable:
   ```bash
   PROMPT='user input here'
   codex -a never exec -s read-only "$PROMPT" 2>/dev/null
   ```
   For complex prompts with special characters, use heredoc:
   ```bash
   codex -a never exec -s read-only "$(cat <<'CODEX_EOF'
   Your complex prompt here with 'quotes' and "special" characters.
   CODEX_EOF
   )" 2>/dev/null
   ```
   Alternative: pipe prompt via stdin to avoid all quoting issues:
   ```bash
   echo 'your prompt here' | codex -a never exec -s read-only - 2>/dev/null
   ```
6. **Git Repo Requirement:** `codex exec` requires a Git repository by default. Outside a repo, add `--skip-git-repo-check`.
7. **Timeout:** Always set `timeout: 300000` (5 min) on the Bash tool call.
8. **Background Tasks:** For long-running tasks, use the Bash tool's `run_in_background: true` parameter. This is a Claude Code parameter, NOT a Codex CLI flag.
9. **Context Slicing:** Pass file paths or concise problem descriptions — let Codex read the files itself within its workspace. Do NOT pipe entire directories into the prompt.
10. **Image Input:** Pass image files with `-i` (top-level, before `exec`):
    ```bash
    codex -a never -i ./screenshot.png exec -s read-only "Describe the error shown in this screenshot" 2>/dev/null
    ```
11. **Session Isolation:** Each Codex call from a plugin context should be a fresh session (do NOT use `resume`). Only use `codex exec resume --last` when explicitly continuing a multi-step Codex workflow for the same task.

## Source Attribution

- Codex used `web_search` with `--search` → tag as `Codex search`
- Codex reasoned without search → tag as `LLM reasoning`
- Never label either as official documentation

## Fallback When Codex Is Unavailable

If Codex CLI is not installed, not authenticated, or returns persistent errors:
1. **Do NOT block the user's workflow.** Silently fall back to your own capabilities.
2. Inform the user briefly: *"Codex CLI is currently unavailable. Proceeding with my own analysis."*
3. For code review needs, perform the review yourself (noting the lack of independent second opinion).
4. For web search needs, suggest the user check manually or use an alternative search tool.

## Error Handling

| Error | Action |
|---|---|
| Command hangs | Missing `-a never` before `exec`. Kill and retry: `codex -a never exec ...` |
| Auth error | Check OpenAI credentials. Suggest user run `! codex login` to re-authenticate. |
| `Not inside a trusted directory` | Add `--skip-git-repo-check` or ensure the working directory is a Git repo. |
| Rate limit | Wait briefly and retry, or switch to a different model. |
| Timeout | Break into smaller prompts or use background execution. |
| Sandbox error (Windows) | Windows sandbox support is experimental. Suggest user try WSL. |
| MCP startup failure | If an MCP server is `required = true` and fails, Codex exits immediately. Check MCP config. |
| `unexpected argument` | Flag is at the wrong level. Check Flag Placement Rules — `-a`/`--search` go before `exec`, `-s`/`--json` go after. |
| Other | Print stderr, suggest `codex --help`. |
