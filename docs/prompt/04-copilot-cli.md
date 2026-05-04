# Copilot CLI 平台 Controller 提示词

## 注入机制

Copilot CLI 使用与 Claude Code 相同的 SessionStart Hook，但走 else 分支输出 SDK 标准格式。

注入源码：`hooks/session-start:53-54`

## 与 Claude Code 的差异

Copilot CLI 的差异也是 JSON 输出格式：

```json
// Claude Code 格式
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "..."
  }
}

// Copilot CLI 格式（SDK 标准）
{
  "additionalContext": "..."
}
```

检测方式：Copilot CLI 设置 `COPILOT_CLI=1` 环境变量。

## 完整提示词

与 Claude Code 完全相同（见 [01-claude-code.md](01-claude-code.md)），包括：

- 完整的 `using-superpowers/SKILL.md` 内容（含 frontmatter）
- 条件性的旧版技能目录警告
- 相同的 `EXTREMELY_IMPORTANT` 包装

## 源码判定逻辑

```bash
# hooks/session-start:49-54
elif [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -z "${COPILOT_CLI:-}" ]; then
  # Claude Code sets CLAUDE_PLUGIN_ROOT without COPILOT_CLI
  printf '{\n  "hookSpecificOutput": { ... }\n' "$session_context"
else
  # Copilot CLI (sets COPILOT_CLI=1) or unknown platform — SDK standard format
  printf '{\n  "additionalContext": "%s"\n}\n' "$session_context"
```

判定优先级：Cursor → Claude Code → Copilot CLI / 未知平台。

## Copilot CLI 特有配置

根据 `docs/README.codex.md`，Copilot CLI 的子智能体功能需要额外配置：

```toml
[features]
multi_agent = true
```

这影响 `dispatching-parallel-agents` 和 `subagent-driven-development` 技能是否能正常工作——没有 multi_agent 功能，无法派遣子智能体。
