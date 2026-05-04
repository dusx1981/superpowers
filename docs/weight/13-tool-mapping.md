# 13 工具映射

工具映射是结构性权重中**最务实的类型**——它不增加权重的语言强度，而是确保技能指令在所有平台上可用。不可用 = 不可执行 = 权重为零。

## 两种工具映射机制

### 13.1 Claude Code → 其他平台

**来源**：`skills/using-superpowers/references/`

```
  ┌──────────────────────────────────────────────────────────┐
  │  映射方向：Claude Code 原生 → 其他平台                   │
  │                                                          │
  │  Claude Code 工具     Copilot CLI 等价         Codex 等价│
  │  ───────────────     ──────────────    ──────────────────│
  │  Skill tool          skill tool        activate_skill    │
  │  TodoWrite           todowrite         todo_write        │
  │  Task (subagent)     @mention          agent dispatch    │
  │  Read/Write/Edit     原生支持          原生支持          │
  │  Bash                原生支持          原生支持          │
  └──────────────────────────────────────────────────────────┘
```

### 13.2 OpenCode 特有映射

**来源**：`.opencode/plugins/superpowers.js:64-71`

```
  ┌──────────────────────────────────────────────────────────┐
  │  映射方向：Claude Code 原生 → OpenCode 等价              │
  │                                                          │
  │  Claude Code          OpenCode                           │
  │  ───────────          ────────                           │
  │  TodoWrite            todowrite                          │
  │  Task (subagent)      OpenCode's subagent system         │
  │  Skill tool           OpenCode's native skill tool       │
  │  Read/Write/Edit/Bash  Native tools                      │
  │                                                          │
  │  注入方式：附加在 bootstrap 内容末尾                     │
  │  注入时机：messages.transform 时                         │
  └──────────────────────────────────────────────────────────┘
```

## 为什么工具映射是权重？

```
  ┌──────────────────────────────────────────────────────────┐
  │  工具映射的逻辑链：                                      │
  │                                                          │
  │  1. 技能引用工具名                                       │
  │     TDD 技能说 "Run: npm test"                           │
  │     subagent-driven-development 说 "Create TodoWrite"    │
  │                                                          │
  │  2. 工具名是平台特定的                                   │
  │     TodoWrite 是 Claude Code 的工具名                    │
  │     OpenCode 的等价工具是 todowrite                      │
  │                                                          │
  │  3. 没有映射 = 工具不可用                                │
  │     AI 看到 "TodoWrite" 但没有这个工具                   │
  │     → 跳过这一步                                        │
  │     → 流程中断                                          │
  │     → 权重归零                                          │
  │                                                          │
  │  4. 有映射 = 工具可用                                    │
  │     AI 看到 "TodoWrite → todowrite"                      │
  │     → 知道该用 todowrite                                │
  │     → 流程继续                                          │
  │     → 权重保持                                          │
  │                                                          │
  │  结论：工具映射不是"翻译"，是"可用性保障"              │
  │  没有映射，所有引用工具的技能都会失效                    │
  └──────────────────────────────────────────────────────────┘
```

## 映射的注入方式对比

```
  Claude Code：
  ────────────
  不需要映射——技能直接使用 Claude Code 原生工具名
  Skill tool、TodoWrite、Task 都是原生工具

  OpenCode：
  ────────
  在 bootstrap 末尾附加映射表：

  "**Tool Mapping for OpenCode:**
   When skills reference tools you don't have,
   substitute OpenCode equivalents:
   - TodoWrite → todowrite
   - Task tool → OpenCode's subagent system
   - Skill tool → OpenCode's native skill tool
   - Read/Write/Edit/Bash → Your native tools"

  Copilot CLI：
  ────────────
  在 references/copilot-tools.md 中定义映射
  AI 需要主动参考这个文件

  Codex：
  ──────
  在 references/codex-tools.md 中定义映射
  使用 <agent-instructions> 标签包裹
  "the model treats tagged blocks as authoritative"

  映射强度的差异：
  - OpenCode：注入到 bootstrap（会话第一条消息）→ 最强
  - Codex：用 XML 标签包裹 → 强
  - Copilot CLI：参考文件（需主动读取）→ 较弱
  - Claude Code：不需要映射 → 最强（原生）
```

## 工具映射的完整性

```
  映射必须覆盖所有被技能引用的工具：

  ┌────────────────────────────────────────────────────────┐
  │  技能中引用的工具         是否需要映射                    │
  │  ──────────────────       ──────────────                │
  │  Skill tool               是（OpenCode: skill tool）   │
  │  TodoWrite                是（OpenCode: todowrite）     │
  │  Task (subagent)          是（OpenCode: subagent）      │
  │  Read                     否（所有平台原生支持）        │
  │  Write                    否（所有平台原生支持）        │
  │  Edit                     否（所有平台原生支持）        │
  │  Bash                     否（所有平台原生支持）        │
  │  Grep/Glob                否（大多数平台原生支持）      │
  │                                                          │
  │  映射不完整 = 部分技能不可用                             │
  │  映射完整 = 所有技能可用                                 │
  │  "可用"是权重的必要条件                                  │
  └────────────────────────────────────────────────────────┘
```

## 映射的维护挑战

```
  ┌──────────────────────────────────────────────────────────┐
  │  工具映射的维护是跨平台技能系统的持续挑战：              │
  │                                                          │
  │  1. 工具名变化                                           │
  │     如果 Claude Code 改了 TodoWrite 的名字               │
  │     所有映射都需要更新                                   │
  │                                                          │
  │  2. 新工具加入                                           │
  │     如果技能体系新增工具引用                              │
  │     映射表需要同步更新                                   │
  │                                                          │
  │  3. 平台差异                                             │
  │     不同平台的工具能力不同                               │
  │     映射可能不是 1:1 的——可能需要适配逻辑               │
  │                                                          │
  │  4. 检测缺失映射                                         │
  │     没有自动化检测"技能引用了未映射的工具"              │
  │     只能在运行时发现（AI 尝试调用不存在的工具）         │
  │                                                          │
  │  OpenCode 的方案（最优）：                               │
  │  在 bootstrap 注入时附加映射表                           │
  │  → 映射表和技能内容在同一个消息中                       │
  │  → AI 不会"忘记查映射"                                 │
  │  → 映射表始终是最新的（因为每次会话都重新读取）         │
  └──────────────────────────────────────────────────────────┘
```
