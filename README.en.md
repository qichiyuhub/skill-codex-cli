# Codex CLI Skill for Claude Code

[中文](README.md) | English

## Purpose

This plugin transforms OpenAI Codex CLI into a proactive collaborator for Claude Code, addressing key limitations through independent reasoning, sandboxed execution, and cross-validation.

## Prerequisites
- **Platform:** Unix-like shell required (macOS, Linux, or Windows with Git Bash/WSL). Windows sandbox support is experimental — WSL is recommended for best experience.
- `codex` CLI installed and available in `PATH` (run `codex --version` to verify).
- OpenAI credentials configured (run `codex login` to authenticate).

## Installation

**Option 1: Plugin Marketplace (recommended)**

```
/plugin marketplace add qichiyuhub/skill-codex-cli
/plugin install skill-codex-cli@skill-codex-cli
```

**Option 2: Manual**

```bash
git clone --depth 1 https://github.com/qichiyuhub/skill-codex-cli.git /tmp/skills-temp
cp -r /tmp/skills-temp/plugins/skill-codex-cli/skills/codex-cli ~/.claude/skills/codex-cli
rm -rf /tmp/skills-temp
```

Restart Claude Code — the skill will be active immediately.

## What Codex Brings to the Table

| Claude's Limitation | Codex's Strength |
|---|---|
| Self-review bias | Independent code reviewer — dedicated `codex review` that doesn't touch the working tree |
| Reasoning uncertainty | Cross-validation from a different model family (OpenAI reasoning models) |
| Knowledge cutoff | Live web search via `--search` flag |
| No sandboxed execution | Three-tier sandbox: `read-only`, `workspace-write`, `danger-full-access` |
| Image analysis limits | Multi-modal: PNG, JPEG via `-i` flag |
| Stuck in logic loop | Fresh-eye review from a different AI perspective |
| Need structured output | `--output-schema` + `--json` for machine-parseable results |
| Need resumable tasks | Session resume/fork for multi-step workflows |

## How It Works

Claude will **proactively** call Codex when it recognizes a task that matches Codex's strengths. No need to say "use Codex" — Claude decides autonomously based on the situation.

### Example Workflows

**Independent code review:**
```
User: Review my changes before I push.
Claude: (delegates to codex review --uncommitted for an unbiased second opinion)
```

**Cross-validation:**
```
User: Is this concurrency logic correct?
Claude: (analyzes first, then asks Codex to independently verify)
```

**Live web search:**
```
User: What are the breaking changes in the latest Next.js release?
Claude: (delegates to Codex with --search for real-time web results)
```

**Structured analysis:**
```
User: Give me a security risk report for this codebase.
Claude: (delegates to Codex with --output-schema for a structured JSON report)
```

## Delegation Modes

| Mode | Command Pattern | Use Case |
|---|---|---|
| Read-only | `codex -a never exec -s read-only` | Analysis, search, code review, planning (recommended default) |
| Workspace write | `codex -a never exec -s workspace-write` | Bug fixes, refactoring, test writing |
| Full autonomy | `codex -a never exec -s danger-full-access` | Only in externally sandboxed environments |

**Note:** `-a` and `--search` are top-level flags (before `exec`); `-s` and `--json` are exec-level flags (after `exec`). Commands can append `2>/dev/null` to suppress stderr noise, but omit it when debugging.

## Troubleshooting

| Problem | Solution |
|---|---|
| `codex: command not found` | Install Codex CLI and ensure it is in your `PATH`. |
| Auth error | Run `! codex login` interactively in Claude Code to authenticate. |
| `Not inside a trusted directory` | Add `--skip-git-repo-check` or ensure working directory is a Git repo. |
| Command hangs | Ensure `-a never` is present **before** `exec` to prevent interactive approval prompts. |
| `unexpected argument` | Flag is at wrong level. `-a`/`--search` go before `exec`, `-s`/`--json` go after. |
| Windows sandbox errors | Windows sandbox is experimental. Use WSL for best experience. |
| Rate limit errors | Wait briefly and retry, or switch to a different model. |

## Skill Reference

Full execution rules, error handling, and collaboration protocol: [`plugins/skill-codex-cli/skills/codex-cli/SKILL.md`](plugins/skill-codex-cli/skills/codex-cli/SKILL.md)

## License

MIT — see [LICENSE](LICENSE).
