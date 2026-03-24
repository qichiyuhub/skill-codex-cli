# Codex CLI Skill for Claude Code

[English](README.en.md) | 中文

## 简介

本插件将 OpenAI Codex CLI 转化为 Claude Code 的主动协作者，通过独立推理、沙箱执行与交叉验证补足关键短板。

## 前置要求
- **平台：** 需要类 Unix Shell 环境（macOS、Linux 或 Windows 的 Git Bash/WSL）。Windows 沙箱支持为实验性功能，推荐使用 WSL 以获得最佳体验。
- 已安装 `codex` CLI 并加入 `PATH`（运行 `codex --version` 验证）。
- 已配置 OpenAI 凭证（运行 `codex login` 完成认证）。

## 安装

**方式一：插件市场（推荐）**

```
/plugin marketplace add qichiyuhub/skill-codex-cli
/plugin install skill-codex-cli@skill-codex-cli
```

**方式二：手动安装**

```bash
git clone --depth 1 https://github.com/qichiyuhub/skill-codex-cli.git /tmp/skills-temp
cp -r /tmp/skills-temp/plugins/skill-codex-cli/skills/codex-cli ~/.claude/skills/codex-cli
rm -rf /tmp/skills-temp
```

重启 Claude Code 后技能即生效。

## Codex 的优势

| Claude 的局限 | Codex 的能力 |
|---|---|
| 自我审查偏差 | 独立代码审查——专用 `codex review`，不影响工作区 |
| 推理不确定性 | 来自不同模型家族（OpenAI 推理模型）的交叉验证 |
| 知识截止日期 | 通过 `--search` 标志实现实时网络搜索 |
| 无沙箱执行 | 三级沙箱：`read-only`、`workspace-write`、`danger-full-access` |
| 图像分析限制 | 多模态支持：通过 `-i` 标志处理 PNG、JPEG |
| 陷入逻辑死循环 | 来自不同 AI 视角的全新审查 |
| 需要结构化输出 | `--output-schema` + `--json` 生成可机器解析的结果 |
| 需要可续期任务 | 会话恢复/分叉支持多步骤工作流 |

## 工作原理

当 Claude 识别到符合 Codex 优势的任务时，会**主动**调用 Codex。无需手动说"使用 Codex"——Claude 会根据情境自主决策。

### 示例工作流

**独立代码审查：**
```
用户：推送前帮我检查一下改动。
Claude：（委托 codex review --uncommitted 进行无偏见的第二意见审查）
```

**交叉验证：**
```
用户：这段并发逻辑正确吗？
Claude：（先自行分析，再请 Codex 独立验证）
```

**实时网络搜索：**
```
用户：最新版 Next.js 有哪些破坏性变更？
Claude：（委托 Codex 携带 --search 标志获取实时网络结果）
```

**结构化分析：**
```
用户：给我生成一份这个代码库的安全风险报告。
Claude：（委托 Codex 携带 --output-schema 生成结构化 JSON 报告）
```

## 委托模式

| 模式 | 命令格式 | 适用场景 |
|---|---|---|
| 只读 | `codex -a never exec -s read-only` | 分析、搜索、代码审查、规划（推荐默认） |
| 工作区写入 | `codex -a never exec -s workspace-write` | Bug 修复、重构、测试编写 |
| 完全自主 | `codex -a never exec -s danger-full-access` | 仅在外部沙箱环境中使用 |

**注意：** `-a` 和 `--search` 是顶级标志（位于 `exec` 之前）；`-s` 和 `--json` 是 exec 级标志（位于 `exec` 之后）。命令可追加 `2>/dev/null` 以抑制 stderr 噪音，但调试时应省略。

## 故障排查

| 问题 | 解决方案 |
|---|---|
| `codex: command not found` | 安装 Codex CLI 并确保其在 `PATH` 中。 |
| 认证错误 | 在 Claude Code 中运行 `! codex login` 进行交互式认证。 |
| `Not inside a trusted directory` | 添加 `--skip-git-repo-check`，或确保工作目录为 Git 仓库。 |
| 命令挂起 | 确保 `-a never` 在 `exec` **之前**，以防止交互式审批提示。 |
| `unexpected argument` | 标志位置错误。`-a`/`--search` 在 `exec` 前，`-s`/`--json` 在 `exec` 后。 |
| Windows 沙箱错误 | Windows 沙箱为实验性功能，推荐使用 WSL。 |
| 限流错误 | 稍等片刻后重试，或切换到其他模型。 |

## 技能参考

完整执行规则、错误处理与协作协议：[`plugins/skill-codex-cli/skills/codex-cli/SKILL.md`](plugins/skill-codex-cli/skills/codex-cli/SKILL.md)

## 许可证

MIT — 详见 [LICENSE](LICENSE)。
