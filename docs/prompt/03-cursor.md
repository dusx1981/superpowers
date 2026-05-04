# Cursor 平台 Controller 提示词

## 注入机制

Cursor 使用与 Claude Code 相同的 SessionStart Hook，但输出 JSON 格式不同。

注入源码：`hooks/session-start:46-48`

## 与 Claude Code 的差异

Cursor 的唯一差异是 JSON 输出格式：

```json
// Claude Code 格式
{
  "hookSpecificOutput": {
    "hookEventName": "SessionStart",
    "additionalContext": "..."
  }
}

// Cursor 格式
{
  "additional_context": "..."
}
```

检测方式：Cursor 设置 `CURSOR_PLUGIN_ROOT` 环境变量（可能同时设置 `CLAUDE_PLUGIN_ROOT`），所以在判断顺序上优先于 Claude Code。

## 完整提示词

与 Claude Code 完全相同（见 [01-claude-code.md](01-claude-code.md)），包括：

- 完整的 `using-superpowers/SKILL.md` 内容（含 frontmatter）
- 条件性的旧版技能目录警告
- 相同的 `EXTREMELY_IMPORTANT` 包装

唯一区别是注入后的 JSON 字段名：`additional_context`（snake_case）vs `hookSpecificOutput.additionalContext`（nested camelCase）。

## 源码判定逻辑

```bash
# hooks/session-start:46-48
if [ -n "${CURSOR_PLUGIN_ROOT:-}" ]; then
  # Cursor sets CURSOR_PLUGIN_ROOT (may also set CLAUDE_PLUGIN_ROOT)
  printf '{\n  "additional_context": "%s"\n}\n' "$session_context"
```

Cursor 在 Claude Code 之前检测，因为它可能同时设置两个环境变量。
