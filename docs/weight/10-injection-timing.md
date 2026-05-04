# 10 注入时机

注入时机是结构性权重中**最隐蔽**的类型——它不改变内容本身，而是改变 AI 何时看到内容。先发制人比后发制人强 10 倍。

## 两种注入机制

### 10.1 Claude Code SessionStart Hook

**来源**：`hooks/session-start`

```
  ┌──────────────────────────────────────────────────────────┐
  │  注入路径：                                              │
  │                                                          │
  │  会话启动                                                │
  │    → session-start Hook 触发                              │
  │    → 读取 using-superpowers/SKILL.md                     │
  │    → JSON 转义                                           │
  │    → 包裹 <EXTREMELY_IMPORTANT>                           │
  │    → 注入为 additionalContext                             │
  │    → 成为 AI 看到的第一个内容                            │
  │                                                          │
  │  平台差异：                                              │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │  Cursor:   additional_context (snake_case)         │  │
  │  │  Claude Code: hookSpecificOutput.additionalContext │  │
  │  │  Copilot CLI: additionalContext (SDK standard)     │  │
  │  └────────────────────────────────────────────────────┘  │
  └──────────────────────────────────────────────────────────┘
```

**权重机制**：

| 维度 | 分析 |
|------|------|
| Hook 类型 | `SessionStart` — 会话启动时触发，不是交互中 |
| 内容位置 | AI 看到的第一条消息——注意力最集中的位置 |
| 格式 | JSON 中的字符串——精确控制每一行 |
| 条件注入 | legacy skills 目录存在时追加 `<important-reminder>` |
| 退出码 | `exit 0` — 即使失败也不影响会话启动 |

### 10.2 OpenCode messages.transform

**来源**：`.opencode/plugins/superpowers.js:101-110`

```
  ┌──────────────────────────────────────────────────────────┐
  │  注入路径：                                              │
  │                                                          │
  │  会话启动                                                │
  │    → messages.transform Hook 触发                        │
  │    → 读取 using-superpowers/SKILL.md                     │
  │    → 剥离 frontmatter                                   │
  │    → 附加 OpenCode 工具映射                              │
  │    → 包裹 <EXTREMELY_IMPORTANT>                           │
  │    → unshift 到第一条用户消息的 parts 前面               │
  │                                                          │
  │  关键代码：                                              │
  │  firstUser.parts.unshift({                               │
  │    ...ref, type: 'text', text: bootstrap                 │
  │  });                                                     │
  │                                                          │
  │  → 注入到用户消息而非系统消息                            │
  │  → unshift = 插入到最前面                                │
  │  → 只注入一次（检查是否已有 EXTREMELY_IMPORTANT）       │
  └──────────────────────────────────────────────────────────┘
```

**权重机制**：

| 维度 | 分析 |
|------|------|
| 注入位置 | 用户消息而非系统消息——避免 token 膨胀 |
| unshift | 插入到 parts 最前面——AI 先读 bootstrap 再读用户输入 |
| 幂等保护 | 检查已有 `EXTREMELY_IMPORTANT`——不会重复注入 |
| frontmatter 剥离 | 去掉 `---` 包裹的 YAML——只保留正文 |
| 工具映射附加 | OpenCode 特有的工具转译——保证技能可用 |

## 为什么注入到用户消息而非系统消息？

```
  ┌──────────────────────────────────────────────────────────┐
  │  superpowers.js 的注释解释了原因：                       │
  │                                                          │
  │  "Using a user message instead of a system message        │
  │   avoids:                                                │
  │   1. Token bloat from system messages                    │
  │      repeated every turn (#750)                          │
  │   2. Multiple system messages breaking                   │
  │      Qwen and other models (#894)"                       │
  │                                                          │
  │  系统消息的问题：                                        │
  │  ────────────────                                        │
  │  - 每轮对话都重复发送 → token 膨胀                      │
  │  - 多条系统消息可能破坏某些模型的处理                    │
  │  - Qwen 等模型对系统消息格式敏感                         │
  │                                                          │
  │  用户消息的方案：                                        │
  │  ────────────────                                        │
  │  - 只注入到第一条用户消息 → 不重复                      │
  │  - 单条消息包含所有 bootstrap → 不破坏格式               │
  │  - 所有模型都支持用户消息 → 兼容性好                    │
  │                                                          │
  │  权重代价：                                              │
  │  - 系统消息 > 用户消息（模型优先级）                    │
  │  - 但用 <EXTREMELY_IMPORTANT> 标签补偿                  │
  │  - 标签权重 > 消息类型差异                              │
  └──────────────────────────────────────────────────────────┘
```

## 注入时机的权重来源

```
  ┌──────────────────────────────────────────────────────────┐
  │  先发制人的三个权重来源：                                │
  │                                                          │
  │  1. 注意力峰值                                           │
  │     ────────────                                         │
  │     AI 处理消息时，开头内容的注意力权重最高              │
  │     SessionStart 注入 = AI 看到的第一个内容              │
  │     → 注意力峰值 + 第一个内容 = 最高权重                 │
  │                                                          │
  │  2. 锚定效应                                             │
  │     ────────                                             │
  │     AI 对后续内容的理解会受开头内容影响                  │
  │     先看到 "You have superpowers" + 铁律                 │
  │     → 后续交互中 AI 会以"我有超能力"为锚点              │
  │     → 不太可能忽略技能体系                               │
  │                                                          │
  │  3. 不可跳过                                             │
  │     ──────────                                           │
  │     用户不能跳过 SessionStart 注入                       │
  │     它在用户输入之前就已经存在                           │
  │     → 用户的第一条消息前面已经有了 bootstrap             │
  │     → AI 处理用户消息时必然先处理 bootstrap              │
  └──────────────────────────────────────────────────────────┘
```

## 两种平台的注入对比

```
  Claude Code (session-start Hook)：
  ──────────────────────────────────
  触发点：会话启动事件
  注入格式：JSON → additionalContext
  消息类型：系统附加上下文
  去重：无（Hook 只触发一次）
  工具映射：不附加（Claude Code 原生支持 Skill 工具）

  OpenCode (messages.transform)：
  ────────────────────────────────
  触发点：消息转换事件
  注入格式：unshift 到用户消息 parts
  消息类型：用户消息（伪装为用户输入）
  去重：检查 EXTREMELY_IMPORTANT 是否已存在
  工具映射：附加 OpenCode 特有的映射表

  共同点：
  - 都注入 <EXTREMELY_IMPORTANT> 包裹的内容
  - 都在 AI 处理用户输入之前完成注入
  - 都只注入一次（幂等性）
  - 都使用 using-superpowers/SKILL.md 作为内容源

  差异点：
  - Claude Code：系统级注入 → 更强但可能 token 膨胀
  - OpenCode：用户消息注入 → 更兼容但优先级略低
  - OpenCode 额外剥离 frontmatter + 附加工具映射
```

## `<important-reminder>` 的条件注入

```
  session-start Hook 中的条件逻辑：

  if [ -d "$legacy_skills_dir" ]; then
      warning_message="<important-reminder>
        IN YOUR FIRST REPLY AFTER SEEING THIS MESSAGE
        YOU MUST TELL THE USER:
        ⚠️ WARNING: Superpowers now uses skills system.
        Move custom skills to ~/.claude/skills instead.
        To make this message go away,
        remove ~/.config/superpowers/skills
      </important-reminder>"
  fi

  权重特征：
  - 条件注入——只在 legacy 目录存在时触发
  - "FIRST REPLY"——精确到"第一条回复"
  - "YOU MUST TELL THE USER"——强制行为
  - "To make this message go away"——告诉 AI 如何消除警告
  - 消退机制——问题解决后不再出现
```
