# 01 注入控制

注入控制是 Superpowers 工作流控制的**最底层**——在 AI 还没看到用户消息之前，就已经被植入了"必须使用技能"的意识。

## 核心机制

```
  用户启动会话
       │
       ▼
  SessionStart Hook / Plugin 触发
       │
       ▼
  读取 using-superpowers/SKILL.md
       │
       ▼
  包裹在 <EXTREMELY_IMPORTANT> 中
       │
       ▼
  注入到 AI 的上下文中（位置因平台而异）
```

## <EXTREMELY_IMPORTANT> 包裹

所有平台的注入内容都被同一个 XML 标签包裹：

```
  <EXTREMELY_IMPORTANT>
  You have superpowers.

  **IMPORTANT: The using-superpowers skill content is included below.
  It is ALREADY LOADED - you are currently following it.
  Do NOT use the skill tool to load "using-superpowers" again
  - that would be redundant.**

  [技能完整内容]

  </EXTREMELY_IMPORTANT>
```

### 为什么用 XML 标签？

XML 标签在 LLM 上下文中有**结构性权重**——比普通文本更难被忽略。`EXTREMELY_IMPORTANT` 这个名字本身就是一种语义强调：

| 包裹方式 | AI 的感知权重 |
|---------|------------|
| 无包裹的普通文本 | 低——可能被当成背景知识 |
| `**粗体**` Markdown | 中——注意到但可以绕过 |
| `<IMPORTANT>` XML 标签 | 高——结构化标记，难以忽略 |
| `<EXTREMELY_IMPORTANT>` XML 标签 | 最高——名称本身就是命令 |

### "ALREADY LOADED" 声明

注入内容明确说"它已经加载——你当前正在遵循它"。这防止 AI 把技能当成"参考资料"而非"必须遵守的指令"。

## 各平台的注入位置

| 平台 | 注入位置 | 技术手段 | 设计理由 |
|------|---------|---------|---------|
| Claude Code | 系统/附加上下文 | `hookSpecificOutput.additionalContext` | 原生 Hook 机制 |
| OpenCode | 首条用户消息的 parts 开头 | `messages.transform` | 避免系统消息 token 膨胀（#750）、避免多系统消息破坏 Qwen 等模型（#894） |
| Cursor | 附加上下文 | `additional_context` | Cursor 的 SDK 标准 |
| Copilot CLI | SDK 标准上下文 | `additionalContext` | Copilot 的 SDK 标准 |

### OpenCode 的用户消息注入为什么有效？

```
  系统消息注入的问题：
  ┌──────────────────────────────────────────────────┐
  │  Turn 1: [system] + [user] + [assistant]          │
  │  Turn 2: [system] + [system] + [user] + [assist]  │  ← 重复
  │  Turn 3: [system] + [system] + [system] + ...     │  ← 膨胀
  └──────────────────────────────────────────────────┘

  用户消息注入的解决方案：
  ┌──────────────────────────────────────────────────┐
  │  Turn 1: [user+bootstrap] + [assistant]           │  ← 只在这一轮
  │  Turn 2: [user] + [assistant]                     │  ← 不重复
  │  Turn 3: [user] + [assistant]                     │  ← 不膨胀
  └──────────────────────────────────────────────────┘
```

## 注入时序

```
  t=0ms    用户打开会话
  t=1ms    平台加载插件系统
  t=2ms    Hook/Plugin 触发
  t=3ms    读取 using-superpowers/SKILL.md（约 117 行）
  t=4ms    构造注入内容（JSON 转义 / frontmatter 剥离）
  t=5ms    注入到上下文
  t=6ms    AI 收到第一条消息时，技能意识已经就位

  用户说的第一个字 → AI 已经在"使用技能"模式下思考
```

## OpenCode 的额外注入：工具映射

OpenCode 版本在技能内容后附加了工具映射段：

```
  **Tool Mapping for OpenCode:**
  When skills reference tools you don't have, substitute OpenCode equivalents:
  - `TodoWrite` → `todowrite`
  - `Task` tool with subagents → Use OpenCode's subagent system (@mention)
  - `Skill` tool → OpenCode's native `skill` tool
  - `Read`, `Write`, `Edit`, `Bash` → Your native tools
```

这是**平台适配层的注入控制**——确保技能中的 Claude Code 工具名称能被正确转译。没有这层映射，AI 在 OpenCode 上找不到 `Skill` 工具会直接报错或忽略技能指令。

## 去重保护

OpenCode 的 `messages.transform` 可能在多次 pass 中被调用。注入代码中有去重检查：

```javascript
// 只在未注入时才注入
if (firstUser.parts.some(p => p.type === 'text' && p.text.includes('EXTREMELY_IMPORTANT'))) return;
```

这防止了 bootstrap 内容被重复注入导致 AI 混乱。

## 源码位置

| 组件 | 文件 |
|------|------|
| Hook 定义（Claude Code/Cursor/Copilot） | `hooks/hooks.json` |
| Hook 脚本 | `hooks/session-start` |
| OpenCode 插件 | `.opencode/plugins/superpowers.js` |
| 被注入的技能内容 | `skills/using-superpowers/SKILL.md` |
