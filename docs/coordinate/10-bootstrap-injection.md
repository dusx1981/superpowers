# 10 模式三：启动注入——会话诞生的上下文构造

会话启动时，插件将 using-superpowers 注入到第一个用户消息中。这是 Controller 的"出生证明"——在用户开口之前，AI 的行为框架就已经被设定了。

## 注入机制：OpenCode 平台

```
  ┌──────────────────────────────────────────────────────────┐
  │  .opencode/plugins/superpowers.js 的注入流程：           │
  │                                                          │
  │  1. 插件注册 'experimental.chat.messages.transform'       │
  │     → 每次对话创建时触发                                 │
  │                                                          │
  │  2. getBootstrapContent() 读取 SKILL.md                  │
  │     → 读取 skills/using-superpowers/SKILL.md             │
  │     → extractAndStripFrontmatter() 剥离 frontmatter      │
  │     → 只保留 body（行为规则）                            │
  │                                                          │
  │  3. 注入工具映射（平台适配层）                           │
  │     → TodoWrite → todowrite                              │
  │     → Task → OpenCode subagent                           │
  │     → Skill → OpenCode skill tool                        │
  │     → Read/Write/Edit/Bash → native tools                │
  │                                                          │
  │  4. 包裹在 EXTREMELY_IMPORTANT 标签中                    │
  │     → "You have superpowers."                            │
  │     → "It is ALREADY LOADED - do NOT load again"         │
  │                                                          │
  │  5. 插入到第一个用户消息的开头                           │
  │     → firstUser.parts.unshift(bootstrap)                 │
  │     → 用户消息内容紧随其后                              │
  │     → AI 先读规则，再读用户输入                          │
  │                                                          │
  │  防重复注入：                                            │
  │  → 检查 firstUser.parts 是否已包含                      │
  │    "EXTREMELY_IMPORTANT"                                 │
  │  → 如果已存在则跳过                                     │
  │  → 防止多轮对话中重复注入                               │
  └──────────────────────────────────────────────────────────┘
```

## 为什么注入到用户消息而非系统消息

```
  ┌──────────────────────────────────────────────────────────┐
  │  superpowers.js 的注释解释了设计决策：                   │
  │                                                          │
  │  "Using a user message instead of a system message        │
  │   avoids:                                                │
  │   1. Token bloat from system messages                    │
  │      repeated every turn (#750)                          │
  │   2. Multiple system messages breaking                   │
  │      Qwen and other models (#894)"                       │
  │                                                          │
  │  问题 1: 系统消息的 token 膨胀                           │
  │  → 某些平台每轮对话都复制系统消息                       │
  │  → 如果 using-superpowers 在系统消息中                  │
  │  → 每轮对话多 ~1000 token                               │
  │  → 10 轮对话 = 多 ~10000 token（浪费）                  │
  │  → 用户消息只出现一次（unshift 到第一条）               │
  │  → 后续对话不会重复                                     │
  │                                                          │
  │  问题 2: 多系统消息导致模型崩溃                         │
  │  → 某些模型（如 Qwen）不支持多个系统消息                │
  │  → 如果平台已有系统消息，再加一个会破坏模型             │
  │  → 用户消息不存在这个问题                               │
  │                                                          │
  │  设计权衡：                                              │
  │  → 系统消息：优先级更高，但有 token 和兼容性问题        │
  │  → 用户消息：优先级略低，但避免了两个关键问题           │
  │  → 用 <EXTREMELY_IMPORTANT> XML 标签弥补优先级差异      │
  │  → 标签的语义强度（"EXTREMELY"、"MUST"、"NOT           │
  │    negotiable"）提升了用户消息中规则的权重               │
  └──────────────────────────────────────────────────────────┘
```

## 启动注入的内容结构

```
  ┌──────────────────────────────────────────────────────────┐
  │  注入到第一个用户消息的完整内容：                        │
  │                                                          │
  │  <EXTREMELY_IMPORTANT>                                   │
  │  You have superpowers.                                   │
  │                                                          │
  │  **IMPORTANT: The using-superpowers skill content is      │
  │   included below. It is ALREADY LOADED - you are          │
  │   currently following it. Do NOT use the skill tool       │
  │   to load "using-superpowers" again - that would be       │
  │   redundant.**                                           │
  │                                                          │
  │  <SUBAGENT-STOP>                                         │
  │  If you were dispatched as a subagent to execute a        │
  │  specific task, skip this skill.                         │
  │  </SUBAGENT-STOP>                                        │
  │                                                          │
  │  <EXTREMELY-IMPORTANT>                                   │
  │  If you think there is even a 1% chance a skill           │
  │  might apply to what you are doing, you ABSOLUTELY        │
  │  MUST invoke the skill.                                  │
  │  ...                                                     │
  │  </EXTREMELY-IMPORTANT>                                  │
  │                                                          │
  │  [using-superpowers SKILL.md body 全文]                   │
  │                                                          │
  │  [平台工具映射]                                          │
  │  </EXTREMELY_IMPORTANT>                                  │
  │                                                          │
  │  三层防护：                                              │
  │  1. <EXTREMELY_IMPORTANT> 外层 — 全局重要性标记          │
  │  2. <SUBAGENT-STOP> 中层 — 子智能体隔离                  │
  │  3. <EXTREMELY-IMPORTANT> 内层 — 1% 技能匹配规则         │
  │                                                          │
  │  信息层次：                                              │
  │  外层 → "你是谁"（有 superpowers 的 AI）                 │
  │  中层 → "你不是谁"（不是子智能体时才生效）              │
  │  内层 → "你怎么做"（1% 原则驱动技能匹配）               │
  │  body → "你做什么"（流程图、红旗表、技能类型...）        │
  │  映射 → "你用什么"（平台工具对照表）                     │
  └──────────────────────────────────────────────────────────┘
```

## 启动注入 vs. 运行时注入的对比

```
  ┌──────────────────────────────────────────────────────────┐
  │  维度          启动注入                    运行时注入    │
  │  ──────        ──────────                  ──────────    │
  │  时机          会话创建时                   用户交互中    │
  │  触发者        插件（自动）                 Controller    │
  │  内容          using-superpowers            其他技能      │
  │  生命周期      整个会话                     技能活跃期    │
  │  隔离度        全局共享                     部分共享      │
  │  可见性        每条消息都可见               加载时可见    │
  │  优先级        最高（EXTREMELY_IMPORTANT）  高            │
  │                                                          │
  │  启动注入 = 宪法                                          │
  │  → 定义了 AI 的基本行为准则                              │
  │  → 整个会话期间有效                                      │
  │  → 其他技能不能违反宪法                                  │
  │                                                          │
  │  运行时注入 = 法律                                        │
  │  → 在特定场景下的具体行为规则                            │
  │  → 在技能活跃期有效                                      │
  │  → 不能违反宪法（用户指令 > 技能 > 默认行为）           │
  └──────────────────────────────────────────────────────────┘
```

## SUBAGENT-STOP 的隔离机制

```
  ┌──────────────────────────────────────────────────────────┐
  │  <SUBAGENT-STOP>                                         │
  │  If you were dispatched as a subagent to execute a        │
  │  specific task, skip this skill.                         │
  │  </SUBAGENT-STOP>                                        │
  │                                                          │
  │  问题：子智能体也看到第一个用户消息                      │
  │  → 如果子智能体也遵循 using-superpowers                  │
  │  → 子智能体也会做技能匹配、流程图检查                   │
  │  → 这与子智能体应"专注单一任务"的设计矛盾              │
  │                                                          │
  │  解决：SUBAGENT-STOP 标签                                 │
  │  → 子智能体读到这个标签后跳过 using-superpowers          │
  │  → 子智能体只遵循 Task prompt 中的指令                   │
  │  → 不做技能匹配，不执行流程图                           │
  │                                                          │
  │  但实际上，子智能体的上下文是 Controller 重建的          │
  │  → 子智能体根本看不到第一个用户消息                      │
  │  → SUBAGENT-STOP 是双保险——                              │
  │    即使子智能体意外看到了启动注入，也会跳过              │
  │                                                          │
  │  两层隔离：                                              │
  │  层 1: Controller 不传递启动注入到子智能体（结构性隔离） │
  │  层 2: SUBAGENT-STOP 让子智能体跳过（规则性隔离）        │
  │  → 两层独立生效，任一层失效都不会导致子智能体            │
  │    误入技能匹配流程                                      │
  └──────────────────────────────────────────────────────────┘
```
