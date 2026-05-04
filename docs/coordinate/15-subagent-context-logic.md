# 15 子智能体上下文的精确构造逻辑

Controller 为每个子智能体构造上下文时，遵循精确的逻辑——不是随意拼接，而是精心选择。本文详细分析三种子智能体的上下文构造输入、处理、输出，以及每个决策背后的理由。

## 实现者子智能体

```
  ┌──────────────────────────────────────────────────────────┐
  │  INPUT:                                                  │
  │    1. plan file path (从 writing-plans 获得)              │
  │    2. current task number (从 TodoWrite 获得)             │
  │    3. work directory (从 using-git-worktrees 获得)        │
  │                                                          │
  │  PROCESS:                                                │
  │    1. 读取计划文件，提取所有任务全文                      │
  │       → "Read plan, extract all tasks with full text"     │
  │       → 一次性提取，不逐任务读取                         │
  │    2. 定位当前任务的全文                                  │
  │       → 粘贴到 "Task Description" 字段                    │
  │    3. 编写 "Scene-setting" 上下文                         │
  │       → Controller 翻译主会话的理解                      │
  │       → "This is Task 3 of 5. Task 1-2 created           │
  │          the parser and validator."                       │
  │    4. 设置工作目录                                        │
  │       → "Work from: /path/to/worktree"                    │
  │                                                          │
  │  OUTPUT:                                                 │
  │    Task prompt = implementer-prompt.md 模板               │
  │      + [Task Description] (粘贴)                          │
  │      + [Context] (Controller 编写)                        │
  │      + [Work from] (worktree 路径)                        │
  │                                                          │
  │  关键决策：为什么不传整个计划文件？                       │
  │  → 计划文件可能很长（所有任务 + 所有代码）               │
  │  → 子智能体只关心当前任务                                │
  │  → 其他任务的代码可能产生干扰                            │
  │  → "don't make subagent read file" = 显式禁止             │
  │  → Controller 是信息中转站，不是文件指针                 │
  │                                                          │
  │  关键决策：为什么要 "Scene-setting"？                     │
  │  → 子智能体不知道自己在整个项目中的位置                  │
  │  → 场景设定让子智能体理解任务间的关系                    │
  │  → 避免"孤岛式实现"——不知道前后依赖                    │
  │  → 但只给必要信息，不给完整对话历史                      │
  │                                                          │
  │  关键决策：为什么允许 "Before You Begin" 提问？           │
  │  → 子智能体可能发现任务描述中的歧义                      │
  │  → 提问通道避免了"猜错方向"的实现                       │
  │  → 提问通过 Controller 中转——子智能体不直接对话用户      │
  │  → Controller 回答后重新派发（带新上下文）               │
  └──────────────────────────────────────────────────────────┘
```

## 规范审阅者子智能体

```
  ┌──────────────────────────────────────────────────────────┐
  │  INPUT:                                                  │
  │    1. 当前任务的规范要求 (从计划文件)                     │
  │    2. 实现者的报告 (从子智能体返回)                       │
  │                                                          │
  │  PROCESS:                                                │
  │    1. 复制任务要求到 "What Was Requested"                  │
  │       → 与实现者看到的 "Task Description" 对齐            │
  │    2. 从实现者报告中提取 "Claims They Built"               │
  │       → 过滤掉思考过程，只保留"声称做了什么"             │
  │    3. 注入 "Do Not Trust" 预设立场                        │
  │       → "suspiciously quickly" = 认知锚点                 │
  │                                                          │
  │  OUTPUT:                                                 │
  │    ✅ Spec compliant                                     │
  │    ❌ Issues: [missing / extra / wrong list]              │
  │                                                          │
  │  关键决策：为什么不传场景设定？                           │
  │  → 场景设定可能让审阅者"理解"实现者的选择               │
  │  → 审阅者应该对照规范检查，不应"理解"设计意图           │
  │  → 理解 = 可能放松标准（共情偏见）                       │
  │  → 严格 = 不理解，只对照                                │
  │                                                          │
  │  关键决策：为什么 "Do Not Trust" 是最重要的框架？         │
  │  → AI 有"讨好倾向"——倾向于同意和信任                   │
  │  → "suspiciously quickly" 预设了不信任的立场             │
  │  → 这不是事实陈述（审阅者不知道实现者花了多久）         │
  │  → 这是一个认知锚点——从怀疑出发，而非信任               │
  │  → 类比：审计师查原始凭证，不只看财务报表               │
  │                                                          │
  │  关键决策：为什么审阅者只做三件事？                       │
  │  → Missing（漏了什么）                                   │
  │  → Extra（多了什么）                                     │
  │  → Misunderstanding（理解错了什么）                      │
  │  → 三个维度覆盖规范偏离的所有可能                       │
  │  → 不评价代码质量——那是代码审阅者的事                   │
  │  → 角色分工的精确执行                                    │
  └──────────────────────────────────────────────────────────┘
```

## 代码审阅者子智能体

```
  ┌──────────────────────────────────────────────────────────┐
  │  INPUT:                                                  │
  │    1. 实现者的报告 (从子智能体返回)                       │
  │    2. 计划文件路径 (从 writing-plans 获得)                │
  │    3. BASE_SHA, HEAD_SHA (从 git 获得)                   │
  │                                                          │
  │  PROCESS:                                                │
  │    1. 填充 code-reviewer.md 模板                          │
  │       → WHAT_WAS_IMPLEMENTED: 实现者报告摘要              │
  │       → PLAN_OR_REQUIREMENTS: Task N from plan            │
  │       → BASE_SHA: 任务开始前的 commit                     │
  │       → HEAD_SHA: 当前 commit                             │
  │       → DESCRIPTION: 一句话摘要                           │
  │    2. 追加 subagent-driven 的额外检查维度                 │
  │       → 文件职责是否清晰                                  │
  │       → 单元是否可独立理解和测试                          │
  │       → 是否遵循计划文件结构                              │
  │       → 是否创建了过大的新文件                            │
  │                                                          │
  │  OUTPUT:                                                 │
  │    Strengths / Issues (Critical/Important/Minor)          │
  │    / Assessment                                           │
  │                                                          │
  │  关键决策：为什么用 SHA 而非文件路径？                    │
  │  → SHA 限定了审阅范围 = 只看本次变更                     │
  │  → 文件路径 = 审阅者可能评价整个文件（范围太宽）         │
  │  → SHA = 精确的 git diff BASE..HEAD                      │
  │  → 范围控制 = 信息最小化原则的体现                       │
  │                                                          │
  │  关键决策：为什么代码审阅在规范审阅之后？                 │
  │  → "Only dispatch after spec compliance review passes."   │
  │  → 先保证做对了（规范匹配），再保证做好了（代码质量）    │
  │  → 如果先代码审阅——可能优化了错误的方向                  │
  │  → 顺序不是随意的——是协议的一部分                       │
  │                                                          │
  │  关键决策：为什么用不同的 subagent_type？                 │
  │  → 实现者/规范审阅者用 general-purpose                    │
  │  → 代码审阅者用 superpowers:code-reviewer                 │
  │  → 代码审阅者有专门的智能体定义（code-reviewer.md）      │
  │  → 不需要从零构造行为规则——模板已定义审阅流程            │
  │  → 审阅维度标准化：Strengths / Issues / Assessment        │
  └──────────────────────────────────────────────────────────┘
```

## 三种子智能体上下文的信息流图

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  计划文件 ──→ Task Full Text ──→ ┌───────────┐          │
  │                                   │ 实现者     │          │
  │  Controller ──→ Scene-setting ──→ │           │          │
  │                                   │ 输出:      │          │
  │  Worktree ──→ Work from ──→      │ DONE/     │          │
  │                                   │ BLOCKED   │          │
  │                                   └─────┬─────┘          │
  │                                         │                │
  │                    实现者报告 ──────────┤                │
  │                                         ↓                │
  │  计划文件 ──→ What Was Requested ─→ ┌───────────┐       │
  │                                      │ 规范审阅者 │       │
  │  实现者报告 ──→ Claims ──→          │           │       │
  │                                      │ 输出:      │       │
  │  Controller ──→ Do Not Trust ──→    │ ✅ / ❌   │       │
  │                                      └─────┬─────┘       │
  │                                            │             │
  │                    实现者报告 ─────────────┤             │
  │                                            ↓             │
  │  git ──→ BASE_SHA, HEAD_SHA ──→ ┌───────────┐          │
  │                                   │ 代码审阅者 │          │
  │  实现者报告 ──→ WHAT_IMPL ──→    │           │          │
  │                                   │ 输出:      │          │
  │  计划文件 ──→ REQUIREMENTS ──→   │ Strengths │          │
  │                                   │ Issues    │          │
  │  Controller ──→ Extra checks ──→ │ Assessment│          │
  │                                   └───────────┘          │
  │                                                          │
  │  信息流的三个特征：                                      │
  │  1. 单向流动——信息从计划/代码/Controller 流向子智能体    │
  │  2. 精确过滤——每种子智能体只看到自己需要的               │
  │  3. 无反馈环——子智能体之间不直接通信                     │
  │     → 实现者不看到审阅者的判断标准                       │
  │     → 审阅者不看到彼此的判断                             │
  │     → Controller 是唯一的信息枢纽                        │
  └──────────────────────────────────────────────────────────┘
```
