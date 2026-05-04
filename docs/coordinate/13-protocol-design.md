# 13 协议设计：技能间的通信契约

Controller 没有代码层面的路由表或消息总线。技能间的通信完全通过**协议**——写在 SKILL.md 中的声明性契约。每个协议定义了三个东西：入口条件、行为约束、出口指向。

## 协议栈：四层通信模型

```
  ┌──────────────────────────────────────────────────────────┐
  │  Layer 4: 会话协议                                       │
  │  ──────────────────                                     │
  │  定义：整个会话的生命周期管理                             │
  │  实现：superpowers.js 插件                               │
  │  内容：                                                 │
  │  - 何时注入 using-superpowers（会话启动时）              │
  │  - 注入到哪里（第一个用户消息）                          │
  │  - 如何避免重复注入（EXTREMELY_IMPORTANT 标记检测）      │
  │  - 如何隔离子智能体（SUBAGENT-STOP 标签）                │
  │                                                          │
  │  Layer 3: 技能路由协议                                   │
  │  ──────────────────                                     │
  │  定义：从用户消息到技能调用的路由规则                     │
  │  实现：using-superpowers SKILL.md                        │
  │  内容：                                                 │
  │  - 匹配规则："Might any skill apply?" → 1% 原则         │
  │  - 优先级：Process skills > Implementation skills        │
  │  - 流程图：从 User message → Skill check → Announce      │
  │  - 红旗表：12 种"跳过技能检查"的理性化及其反驳          │
  │                                                          │
  │  Layer 2: 技能切换协议                                   │
  │  ──────────────────                                     │
  │  定义：技能之间的转移规则                                 │
  │  实现：每个技能 SKILL.md 内的唯一出口声明                 │
  │  内容：                                                 │
  │  - 唯一出口：brainstorming → writing-plans               │
  │  - 分支出口：writing-plans → subagent | executing        │
  │  - 终态出口：subagent → finishing                        │
  │  - 横切入口：debugging / verification 的触发条件         │
  │                                                          │
  │  Layer 1: 子智能体通信协议                               │
  │  ──────────────────                                     │
  │  定义：Controller 与子智能体之间的消息格式                │
  │  实现：implementer-prompt.md / spec-reviewer-prompt.md    │
  │  内容：                                                 │
  │  - 输入格式：Task Description / Context / Before Begin   │
  │  - 输出格式：DONE / BLOCKED / NEEDS_CONTEXT / CONCERNS   │
  │  - 审阅格式：✅ / ❌ + Issues list                       │
  │  - 重试协议：审阅失败 → 修复 → 重新审阅                 │
  │                                                          │
  │  四层协议的依赖关系：                                    │
  │  Layer 4 提供 Layer 3 的运行环境                         │
  │  Layer 3 调用 Layer 2 的路由规则                         │
  │  Layer 2 触发 Layer 1 的子智能体派发                     │
  │  → 自上而下的依赖，自下而上的执行                        │
  └──────────────────────────────────────────────────────────┘
```

## 唯一出口协议：详细规范

```
  ┌──────────────────────────────────────────────────────────┐
  │  协议名称：唯一出口协议 (Single Exit Protocol)            │
  │  ─────────────────────────────────────────────           │
  │                                                          │
  │  规则：每个技能在 SKILL.md 中声明自己的出口指向           │
  │  → 出口指向是唯一的（或有限分支）                        │
  │  → AI 没有选择跳到其他技能的裁量空间                     │
  │  → 出口声明是不可协商的                                  │
  │                                                          │
  │  协议语法（出现在 SKILL.md 中）：                        │
  │                                                          │
  │  brainstorming:                                          │
  │    "Do NOT invoke any other skill.                        │
  │     writing-plans is the next step."                      │
  │    → 严格唯一出口                                        │
  │    → 禁止调用任何其他技能                                │
  │                                                          │
  │  writing-plans:                                          │
  │    "REQUIRED SUB-SKILL: Use                               │
  │     superpowers:subagent-driven-development (recommended) │
  │     or superpowers:executing-plans"                       │
  │    → 分支出口（2 选 1）                                  │
  │    → 有推荐标记（降低决策难度）                          │
  │                                                          │
  │  subagent-driven-development:                             │
  │    流程图终态节点 → "Use                                  │
  │     superpowers:finishing-a-development-branch"           │
  │    → 隐式唯一出口（在流程图中定义）                      │
  │                                                          │
  │  协议保证：                                              │
  │  1. 没有技能会"迷路"——每个技能都知道下一步              │
  │  2. 没有技能会"越权"——出口限制在声明范围内              │
  │  3. 没有技能会"死循环"——出口总是指向下一个技能           │
  │  4. 没有技能会"跳过"——唯一出口是强制的                  │
  │                                                          │
  │  协议违约：                                              │
  │  如果 AI 违反出口协议（如 brainstorming 后直接实现）：   │
  │  → HARD-GATE 拦截（"Do NOT write any code"）              │
  │  → 红旗表拦截（"This doesn't need a formal skill"）      │
  │  → 用户可以纠正（"先做设计"）                           │
  │  → 三重保障，出口协议的违约概率极低                      │
  └──────────────────────────────────────────────────────────┘
```

## 横切触发协议：动态路由规则

```
  ┌──────────────────────────────────────────────────────────┐
  │  协议名称：横切触发协议 (Cross-Cut Trigger Protocol)      │
  │  ─────────────────────────────────────────────           │
  │                                                          │
  │  横切技能不走唯一出口——它们由运行时条件触发              │
  │  → 触发条件写在技能的 description 中                     │
  │  → Controller 在每条消息时检查触发条件                   │
  │                                                          │
  │  触发条件语法：                                          │
  │                                                          │
  │  systematic-debugging:                                    │
  │    "Use when encountering any bug, test failure,          │
  │     or unexpected behavior, before proposing fixes"       │
  │    → 触发信号：bug / test failure / unexpected            │
  │    → 触发时机：在提出修复之前                            │
  │                                                          │
  │  verification-before-completion:                          │
  │    "Use when about to claim work is complete, fixed,      │
  │     or passing, before committing or creating PRs"        │
  │    → 触发信号：声称完成/修复/通过                        │
  │    → 触发时机：在提交/PR 之前                            │
  │                                                          │
  │  receiving-code-review:                                   │
  │    "Use when receiving code review feedback,              │
  │     before implementing suggestions"                      │
  │    → 触发信号：收到审阅反馈                              │
  │    → 触发时机：在实施建议之前                            │
  │                                                          │
  │  横切触发 vs. 唯一出口：                                  │
  │  ────────────────────────                                │
  │  唯一出口：编译时确定（静态路由）                        │
  │  → 写在 SKILL.md 中，不随运行时变化                     │
  │  → 类比：高速公路上的固定出口                            │
  │                                                          │
  │  横切触发：运行时确定（动态路由）                        │
  │  → 依赖当前对话的上下文                                  │
  │  → 类比：高速公路上的应急匝道                           │
  │                                                          │
  │  两者的交互：                                            │
  │  → 主链按唯一出口推进                                    │
  │  → 横切技能插入主链，处理后回到主链                     │
  │  → 主链不会因为横切触发而改变终点                      │
  │  → 横切触发只影响主链的中间状态                         │
  │                                                          │
  │  触发顺序的冲突解决：                                    │
  │  如果同时满足多个横切触发条件：                          │
  │  → Skill Priority 规则：Process skills first             │
  │  → debugging (Process) > verification (Process)          │
  │  → Process skills > Implementation skills                │
  │  → 内置优先级消除了触发冲突                             │
  └──────────────────────────────────────────────────────────┘
```

## 子智能体通信协议：消息格式规范

```
  ┌──────────────────────────────────────────────────────────┐
  │  协议名称：子智能体通信协议                               │
  │           (Subagent Communication Protocol)               │
  │  ─────────────────────────────────────────────           │
  │                                                          │
  │  协议定义了 Controller 与子智能体之间的消息格式：         │
  │                                                          │
  │  ┌─ 请求消息（Controller → 子智能体）─────────────────┐  │
  │  │                                                     │  │
  │  │  Task tool 参数：                                    │  │
  │  │  - description: 任务摘要（1 行）                     │  │
  │  │  - prompt: 完整指令（多行）                          │  │
  │  │    - 角色：You are [implementing/reviewing/...]      │  │
  │  │    - 输入数据：[Full text / Report / SHA range]      │  │
  │  │    - 行为规则：[Your Job / Self-Review / Escalate]   │  │
  │  │    - 输出格式：[Report Format / ✅/❌]               │  │
  │  │                                                     │  │
  │  │  三种子智能体的请求消息差异：                         │  │
  │  │                                                     │  │
  │  │  实现者：                                            │  │
  │  │    subagent_type: general-purpose                    │  │
  │  │    必需字段：Task Description, Context, Work from    │  │
  │  │    可选字段：Before You Begin (提问通道)             │  │
  │  │                                                     │  │
  │  │  规范审阅者：                                        │  │
  │  │    subagent_type: general-purpose                    │  │
  │  │    必需字段：What Was Requested, Claims, Do Not Trust │  │
  │  │    禁止字段：Context (场景设定)                      │  │
  │  │                                                     │  │
  │  │  代码审阅者：                                        │  │
  │  │    subagent_type: superpowers:code-reviewer          │  │
  │  │    必需字段：WHAT_WAS_IMPLEMENTED, BASE_SHA, HEAD_SHA│  │
  │  │    模板引用：code-reviewer.md                        │  │
  │  └─────────────────────────────────────────────────────┘  │
  │                                                          │
  │  ┌─ 响应消息（子智能体 → Controller）─────────────────┐  │
  │  │                                                     │  │
  │  │  实现者的响应格式：                                  │  │
  │  │    Status: DONE | DONE_WITH_CONCERNS | BLOCKED       │  │
  │  │           | NEEDS_CONTEXT                            │  │
  │  │    What implemented: [描述]                          │  │
  │  │    Test results: [测试结果]                          │  │
  │  │    Files changed: [文件列表]                         │  │
  │  │    Self-review findings: [自审发现]                  │  │
  │  │    Issues/concerns: [问题]                           │  │
  │  │                                                     │  │
  │  │  审阅者的响应格式：                                  │  │
  │  │    规范审阅：✅ Spec compliant                       │  │
  │  │              ❌ Issues: [missing/extra/wrong list]    │  │
  │  │    代码审阅：Strengths / Issues (C/I/M) / Assessment │  │
  │  │                                                     │  │
  │  │  Controller 对响应的处理：                           │  │
  │  │    DONE → 进入审阅                                   │  │
  │  │    DONE_WITH_CONCERNS → 评估关切后决定               │  │
  │  │    BLOCKED → 评估阻塞后重新派发                      │  │
  │  │    NEEDS_CONTEXT → 提供上下文后重新派发              │  │
  │  │    ✅ → 进入下一个审阅阶段                           │  │
  │  │    ❌ → 实现者修复 → 重新审阅                       │  │
  │  └─────────────────────────────────────────────────────┘  │
  │                                                          │
  │  关键设计：                                              │
  │  → 响应格式是结构化的（Status 字段是枚举值）            │
  │  → 不是自由文本——Controller 可以精确解析状态            │
  │  → 但具体描述是自由文本——子智能体可以详细说明           │
  │  → 结构化 + 自由文本 = 可解析 + 可理解                  │
  └──────────────────────────────────────────────────────────┘
```

## 协议的版本演进

```
  ┌──────────────────────────────────────────────────────────┐
  │  协议是 Markdown 文件，不是编译代码                       │
  │  → 修改协议 = 修改 SKILL.md                              │
  │  → 不需要重新编译、重新部署、重新启动                    │
  │  → 下一个会话自动使用新协议                              │
  │                                                          │
  │  但这也意味着：                                          │
  │  → 协议没有版本号——修改即生效                           │
  │  → 协议没有回滚——只能 git revert                        │
  │  → 协议没有测试——只能人工验证                           │
  │  → 协议没有类型检查——拼写错误可能破坏协议              │
  │                                                          │
  │  风险缓解：                                              │
  │  1. 协议的冗余设计——关键约束在多处重复                  │
  │  2. 协议的声明式风格——AI 对措辞变化有容错能力           │
  │  3. 协议的自审清单——writing-plans 的自审可以              │
  │     检测部分协议破坏                                     │
  │  4. 人工验证——brainstorming 的 User Review Gate           │
  │     和 finishing 的测试验证是人工把关点                  │
  │                                                          │
  │  协议演进的例子：                                        │
  │  如果新增一个技能 "security-review"：                    │
  │  → 不需要修改 using-superpowers（1% 原则自动匹配）       │
  │  → 只需要写 security-review/SKILL.md                     │
  │  → 在 subagent-driven-development 中声明                 │
  │    "Subagents should use: superpowers:security-review"    │
  │  → 其他技能不受影响                                     │
  │  → 新增技能 = 纯增量修改，无扩散效应                    │
  └──────────────────────────────────────────────────────────┘
```
