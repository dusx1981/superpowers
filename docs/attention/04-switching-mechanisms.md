# 04 注意力切换：技能转换时的焦点重定向

当 AI 从一个技能切换到另一个技能时，旧技能的规则仍在上下文中，新技能的规则刚被加载。此时注意力处于"不稳定状态"——两套规则竞争同一个注意力预算。Superpowers 用三种机制确保注意力正确切换。

## 问题：注意力切换时的竞争

```
  ┌──────────────────────────────────────────────────────────┐
  │  brainstorming → writing-plans 切换时：                  │
  │                                                          │
  │  上下文中同时存在：                                      │
  │  - brainstorming 的 HARD-GATE: "不要写代码"              │
  │  - brainstorming 的清单: "第 9 步: invoke writing-plans"  │
  │  - writing-plans 的 Overview: "写实现计划"               │
  │  - writing-plans 的任务结构: "Write the failing test"     │
  │                                                          │
  │  竞争：                                                  │
  │  brainstorming 说"不要实现"                              │
  │  writing-plans 说"写实现计划（包含代码）"                │
  │  → AI 可能困惑："我要不要写代码？"                      │
  │  → 如果 brainstorming 的 HARD-GATE 仍占主导：            │
  │    AI 可能拒绝写计划中的代码示例                         │
  │  → 如果 writing-plans 的规则占了主导：                   │
  │    AI 可能跳过计划直接实现                               │
  │  → 两种错误都很可能发生                                  │
  │                                                          │
  │  切换的不稳定窗口：                                      │
  │  ┌──────────────────────────────────────────────────┐    │
  │  │  时间线                                           │    │
  │  │  ──────                                           │    │
  │  │  T1: brainstorming 完成 → 稳定状态                │    │
  │  │  T2: writing-plans 加载 → 不稳定窗口（竞争）      │    │
  │  │  T3: AI 开始执行 writing-plans → 新稳定状态        │    │
  │  │                                                     │    │
  │  │  T2 是最危险的窗口：                               │    │
  │  │  → 两套规则同时激活                               │    │
  │  │  → AI 不确定哪套规则优先                           │    │
  │  │  → 行为可能偏向任何一方                            │    │
  │  │  → 需要机制帮助 AI 快速通过 T2                    │    │
  │  └──────────────────────────────────────────────────┘    │
  └──────────────────────────────────────────────────────────┘
```

## 机制 1: 唯一出口——切换的"信号灯"

```
  ┌──────────────────────────────────────────────────────────┐
  │  唯一出口 = 旧技能的"遗言"                              │
  │                                                          │
  │  brainstorming 的最后一步：                              │
  │  "Invoke the writing-plans skill to create a              │
  │   detailed implementation plan.                           │
  │   Do NOT invoke any other skill.                          │
  │   writing-plans is the next step."                        │
  │                                                          │
  │  效果：                                                  │
  │  → AI 的最后一条活跃指令 = "调用 writing-plans"          │
  │  → 这条指令成为注意力的焦点                            │
  │  → AI 执行 Skill("writing-plans")                        │
  │  → writing-plans 的内容被加载到上下文                    │
  │  → AI 的注意力自然被新内容吸引                          │
  │  → 旧规则的权重被新规则的加载事件降低                   │
  │                                                          │
  │  唯一出口的"信号灯"功能：                               │
  │  → 绿灯：明确指向下一个技能                             │
  │  → 红灯：禁止调用其他技能                               │
  │  → AI 不需要"判断"该切换到哪个技能                    │
  │  → 切换路径已经确定                                     │
  │  → 不稳定窗口被缩短（AI 不需要犹豫）                   │
  └──────────────────────────────────────────────────────────┘
```

## 机制 2: 宣布仪式——切换的"确认信号"

```
  ┌──────────────────────────────────────────────────────────┐
  │  宣布仪式发生在切换之后、执行之前：                      │
  │                                                          │
  │  Skill("writing-plans") 加载完成                          │
  │  → AI 宣布："I'm using the writing-plans skill            │
  │     to create the implementation plan."                   │
  │  → 宣布进入对话历史 → 获得最高注意力权重                │
  │  → 宣布中包含技能名称和目的                              │
  │  → 注意力焦点从"切换"转移到"执行"                      │
  │                                                          │
  │  宣布仪式如何消除竞争：                                  │
  │  → "I'm using writing-plans" = 我现在执行 writing-plans   │
  │  → = brainstorming 的规则不再是我的活跃规则集             │
  │  → = brainstorming 的 HARD-GATE 仍存在但我已不在         │
  │     brainstorming 阶段                                   │
  │  → 宣布 = 注意力的"频道切换"                           │
  │  → 类比：电视遥控器——从频道 1 切换到频道 2              │
  │  → 频道 1 的信号仍在但不再是焦点                        │
  │                                                          │
  │  哪些技能有宣布仪式：                                    │
  │  writing-plans ✓   brainstorming ✗                       │
  │  using-git-worktrees ✓  TDD ✗                            │
  │  finishing ✓       debugging ✗                           │
  │  executing-plans ✓  verification ✗                       │
  │                                                          │
  │  规律：协调技能有宣布仪式，纪律技能没有                 │
  │  → 协调技能（writing-plans, finishing）需要切换注意力    │
  │  → 纪律技能（TDD, debugging）需要保持注意力             │
  │  → 切换型技能 = 有宣布                                 │
  │  → 纪律型技能 = 有铁律                                 │
  │  → 两种机制服务于不同类型的注意力需求                   │
  └──────────────────────────────────────────────────────────┘
```

## 机制 3: 新技能的"开场重锤"——重新锚定注意力

```
  ┌──────────────────────────────────────────────────────────┐
  │  新技能加载后，AI 的前几行看到的内容                    │
  │  决定了注意力的新锚点。                                 │
  │                                                          │
  │  写计划的"开场"：                                      │
  │  "Write comprehensive implementation plans                │
  │   assuming the engineer has zero context                  │
  │   for our codebase and questionable taste."               │
  │  → 角色设定："我是为无知工程师写计划的人"               │
  │  → "questionable taste" = 不信任实现者的设计判断         │
  │  → 注意力锚定在"详细"和"完整"上                       │
  │                                                          │
  │  subagent-driven 的"开场"：                              │
  │  "Execute plan by dispatching fresh subagent              │
  │   per task, with two-stage review after each."            │
  │  → 角色设定："我是派发子智能体的协调者"                 │
  │  → 注意力锚定在"派发"和"审阅"上                       │
  │                                                          │
  │  对比 brainstorming 的"开场"：                            │
  │  HARD-GATE 在第一行就锚定了注意力                       │
  │  → "不要写代码" = 最强的开场锚定                        │
  │  → 后续所有行为都在这个锚定下运行                       │
  │                                                          │
  │  开场 = 注意力的新起点                                   │
  │  → 如果开场是约束（HARD-GATE）→ 注意力被约束锚定       │
  │  → 如果开场是角色（writing-plans）→ 注意力被角色锚定    │
  │  → 如果开场是原则（subagent-driven）→ 注意力被原则锚定  │
  │  → 任何强开场都比弱开场更好地锚定注意力                 │
  │  → 最弱的开场是模糊的描述                               │
  │  → 最强的开场是绝对禁止                                 │
  └──────────────────────────────────────────────────────────┘
```
