# 09 Prompt 模板

Prompt 模板是结构性权重中**最具约束力**的类型——为子智能体提供预制的角色、指令和输出格式，子智能体不是"参考"模板，而是"遵循"模板。

## 3 套 Prompt 模板

### 9.1 实现者 Prompt 模板

**来源**：`skills/subagent-driven-development/implementer-prompt.md`

**模板结构**：

```
  ┌──────────────────────────────────────────────────────────┐
  │  You are implementing Task N: [task name]                │
  │                                                          │
  │  ## Task Description                                     │
  │  [FULL TEXT — 不让子智能体读文件]                        │
  │                                                          │
  │  ## Context                                              │
  │  [Scene-setting: 哪里、依赖、架构上下文]                 │
  │                                                          │
  │  ## Before You Begin                                     │
  │  [有问题先问，不要猜]                                    │
  │                                                          │
  │  ## Your Job                                             │
  │  1-6 步：实现、测试、验证、提交、自审、报告              │
  │                                                          │
  │  ## Code Organization                                    │
  │  [文件职责、单一责任、遵循既有模式]                      │
  │                                                          │
  │  ## When You're in Over Your Head                        │
  │  ["停止并升级"机制]                                      │
  │                                                          │
  │  ## Before Reporting Back: Self-Review                   │
  │  [完整性、质量、纪律、测试四维度自审]                    │
  │                                                          │
  │  ## Report Format                                        │
  │  Status: DONE | DONE_WITH_CONCERNS | BLOCKED |           │
  │          NEEDS_CONTEXT                                    │
  │  [具体报告字段]                                          │
  └──────────────────────────────────────────────────────────┘
```

**权重机制**：

| 部分 | 权重 |
|------|------|
| "You are implementing" | 角色定义——子智能体知道自己的身份 |
| "FULL TEXT" | 上下文完整——不需要自己读文件 |
| "Before You Begin" | 提问优先——不允许猜测 |
| "When You're in Over Your Head" | 升级机制——不允许硬撑 |
| "Bad work is worse than no work" | 道德约束——做得差比不做更糟 |
| Report Format | 结构化输出——不允许模糊报告 |

### 9.2 规范审阅者 Prompt 模板

**来源**：`skills/subagent-driven-development/spec-reviewer-prompt.md`

**模板结构**：

```
  ┌──────────────────────────────────────────────────────────┐
  │  You are reviewing whether an implementation             │
  │  matches its specification.                              │
  │                                                          │
  │  ## What Was Requested                                   │
  │  [FULL TEXT of task requirements]                        │
  │                                                          │
  │  ## What Implementer Claims They Built                   │
  │  [From implementer's report]                             │
  │                                                          │
  │  ## CRITICAL: Do Not Trust the Report                    │
  │  DO NOT:                                                 │
  │  - Take their word for what they implemented             │
  │  - Trust their claims about completeness                 │
  │  - Accept their interpretation of requirements           │
  │                                                          │
  │  DO:                                                     │
  │  - Read the actual code                                  │
  │  - Compare line by line                                  │
  │  - Check for missing pieces                              │
  │  - Look for extra features                               │
  │                                                          │
  │  ## Your Job                                             │
  │  Missing / Extra / Misunderstandings                     │
  │                                                          │
  │  Report:                                                 │
  │  ✅ Spec compliant / ❌ Issues found                     │
  └──────────────────────────────────────────────────────────┘
```

**权重机制**：

| 部分 | 权重 |
|------|------|
| "Do Not Trust the Report" | 不信任原则——审阅者和实现者利益对立 |
| "CRITICAL" | 最高级标注——不可忽视 |
| "DO NOT" / "DO" | 否定/肯定对称——同时禁止和命令 |
| "line by line" | 审查粒度——不允许粗略审查 |
| "Read the actual code" | 证据来源——不看报告看代码 |
| "finished suspiciously quickly" | 预判——假定实现者可能偷懒 |

### 9.3 代码质量审阅者 Prompt 模板

**来源**：`skills/subagent-driven-development/code-quality-reviewer-prompt.md`、`skills/requesting-code-review/code-reviewer.md`

**模板结构**：

```
  ┌──────────────────────────────────────────────────────────┐
  │  Use template at requesting-code-review/code-reviewer.md │
  │                                                          │
  │  WHAT_WAS_IMPLEMENTED: [from implementer's report]       │
  │  PLAN_OR_REQUIREMENTS: Task N from [plan-file]           │
  │  BASE_SHA: [commit before task]                          │
  │  HEAD_SHA: [current commit]                              │
  │  DESCRIPTION: [task summary]                             │
  │                                                          │
  │  Additional checks:                                      │
  │  - Each file one clear responsibility?                   │
  │  - Units independently testable?                         │
  │  - Following plan file structure?                        │
  │  - New files already large?                              │
  │                                                          │
  │  Returns: Strengths, Issues (C/I/M), Assessment          │
  └──────────────────────────────────────────────────────────┘
```

**权重机制**：

| 部分 | 权重 |
|------|------|
| BASE_SHA / HEAD_SHA | 精确范围——审阅者只看本次变更 |
| "Don't flag pre-existing file sizes" | 范围限定——不归因于前人 |
| Critical/Important/Minor | 三级严重度——不允许一切问题同等 |
| "Only dispatch after spec compliance passes" | 顺序锁——必须先过规范审 |

## Prompt 模板 vs 自由调度的权重差异

```
  自由调度（弱）：
  ────────────────
  Controller: "帮我实现这个任务"
  → 子智能体自行决定如何报告、何时提问、如何自审

  Prompt 模板（强）：
  ──────────────────
  Controller: [用模板调度，包含 8 个预定义章节]
  → 子智能体遵循模板的每个章节
  → 报告格式固定（4 种状态码）
  → 提问时机固定（开始前）
  → 自审维度固定（4 个维度）
  → 升级机制固定（BLOCKED/NEEDS_CONTEXT）

  差异：
  - 自由调度：子智能体有 100% 裁量空间
  - Prompt 模板：子智能体只在模板未规定的部分有裁量空间
  - 模板把"应该做的"变成了"必须做的"
```

## "Don't make subagent read plan file"

```
  ┌──────────────────────────────────────────────────────────┐
  │  实现者模板的指令：                                      │
  │                                                          │
  │  "[FULL TEXT of task from plan -                         │
  │   paste it here, don't make subagent read file]"         │
  │                                                          │
  │  为什么不让子智能体自己读文件？                          │
  │                                                          │
  │  1. 上下文隔离                                           │
  │     子智能体是新实例，没有会话上下文                     │
  │     读文件可能读到不相关的内容                           │
  │                                                          │
  │  2. 精确控制                                             │
  │     Controller 粘贴的是"这个任务需要的全部信息"          │
  │     子智能体读文件可能读到其他任务的信息                 │
  │                                                          │
  │  3. 效率                                                 │
  │     读文件消耗 token 和时间                              │
  │     粘贴文本直接可用                                     │
  │                                                          │
  │  4. 防止误解                                             │
  │     Controller 可以在粘贴时调整措辞                      │
  │     子智能体读文件可能理解错误                           │
  │                                                          │
  │  Red Flags 也明确禁止：                                  │
  │  "Never make subagent read plan file                     │
  │   (provide full text instead)"                           │
  └──────────────────────────────────────────────────────────┘
```

## Prompt 模板的"不可变"区域

```
  模板中有些部分是固定的（不可变），有些是参数化的（可变）：

  不可变部分：
  ────────────
  - 章节结构（8 个章节必须都在）
  - 状态码（4 种，不能自创）
  - 自审维度（4 个，不能增减）
  - 升级机制（BLOCKED/NEEDS_CONTEXT）
  - "Bad work is worse than no work"

  可变部分：
  ────────
  - Task Description（每个任务不同）
  - Context（每个任务不同）
  - Work directory（每个任务不同）
  - 模型选择（根据复杂度选择）

  设计原则：
  - 不可变部分 = 流程控制 → 模板保证
  - 可变部分 = 任务内容 → Controller 注入
  - 子智能体在"流程"上没有裁量权
  - 子智能体在"内容"上有执行权
```
