# 09 模式二：子智能体派发——隔离的上下文构造

子智能体派发是 Controller 最精密的上下文构造——每个子智能体获得一个全新的、独立的上下文，与主会话完全隔离。Controller 不是"传递信息"，而是"重建信息"。

## 派发的隔离原理

```
  ┌──────────────────────────────────────────────────────────┐
  │  主会话上下文（Controller 看到的世界）：                 │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ using-superpowers 规则                             │  │
  │  │ brainstorming 历史（用户问答、设计决策）            │  │
  │  │ writing-plans 历史（实现计划、任务列表）            │  │
  │  │ subagent-driven 规则 + 之前任务的执行结果           │  │
  │  │ 当前任务的实现者报告                               │  │
  │  │ 之前审阅者的反馈                                   │  │
  │  │ ... 可能上百条消息 ...                              │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  实现者子智能体上下文（新建的干净世界）：                 │
  │  ┌────────────────────────────────────────────────────┐  │
  │  │ [Task description] ← 仅此任务                      │  │
  │  │ [Task full text]   ← Controller 手动复制            │  │
  │  │ [Scene-setting]    ← Controller 手动编写             │  │
  │  │ [Behavior rules]   ← implementer-prompt.md 模板     │  │
  │  │                                                     │  │
  │  │ 看不到：                                           │  │
  │  │ ✗ 之前的任务实现                                   │  │
  │  │ ✗ 其他审阅者的反馈                                 │  │
  │  │ ✗ 用户的设计偏好                                   │  │
  │  │ ✗ using-superpowers 的全局规则                     │  │
  │  │ ✗ brainstorming 的问答历史                         │  │
  │  └────────────────────────────────────────────────────┘  │
  │                                                          │
  │  关键差异：                                              │
  │  主会话可能有 50+ 条消息，上千个 token 的历史             │
  │  子智能体只有 Controller 精心挑选的几百个 token           │
  │  → 信息密度极高，噪音为零                               │
  └──────────────────────────────────────────────────────────┘
```

## 三种子智能体的上下文对比

```
  ┌──────────────────────────────────────────────────────────┐
  │            实现者        规范审阅者      代码审阅者      │
  │  ────────  ────────      ──────────      ──────────      │
  │  角色      实现任务      检查规范匹配    检查代码质量    │
  │                                                          │
  │  输入                                                │
  │  ────                                                 │
  │  任务全文   ✓ (粘贴)      ✓ (粘贴)        ✗ (不传入)   │
  │  场景设定   ✓ (编写)      ✗ (不传入)      ✗ (不传入)   │
  │  实现者报告 ✗ (无)        ✓ (从报告提取)  ✓ (从报告)   │
  │  规范文本   ✓ (隐含在      ✓ (粘贴)        ✗ (不传入)   │
  │             任务中)                                      │
  │  代码 diff  ✗ (自己写)     ✓ (读实际代码)  ✓ (git range)│
  │  BASE_SHA   ✗ (不传入)     ✗ (不传入)      ✓ (必需)    │
  │  HEAD_SHA   ✗ (不传入)     ✗ (不传入)      ✓ (必需)    │
  │                                                          │
  │  行为约束                                            │
  │  ────────                                            │
  │  自审       ✓ (强制的)     ✗ (不信任       ✗ (标准     │
  │                           原则)           审阅模板)    │
  │  TDD        ✓ (推荐)       ✗ (无关)        ✗ (无关)   │
  │  不信任     ✗ (无关)       ✓ (核心原则)    ✗ (审阅    │
  │                                          职责)          │
  │  YAGNI      ✓ (强制的)     ✓ (查多余)      ✗ (可选)   │
  │                                                          │
  │  关键观察：                                              │
  │  → 三种子智能体看到的信息完全不同                        │
  │  → 实现者不知道审阅者的存在                              │
  │  → 规范审阅者不知道代码审阅者的存在                      │
  │  → 代码审阅者不知道规范审阅者的判断                      │
  │  → 每个子智能体在自己的信息孤岛中独立判断                │
  └──────────────────────────────────────────────────────────┘
```

## 实现者子智能体的上下文构造

```
  ┌──────────────────────────────────────────────────────────┐
  │  Controller 派发实现者时的 Task prompt 结构：             │
  │                                                          │
  │  Task tool (general-purpose):                            │
  │    description: "Implement Task N: [task name]"           │
  │    prompt: |                                             │
  │      You are implementing Task N: [task name]            │
  │                                                          │
  │      ## Task Description                                 │
  │      [FULL TEXT of task from plan -                      │
  │       paste it here, don't make subagent read file]      │
  │                                                          │
  │      ## Context                                          │
  │      [Scene-setting: where this fits,                    │
  │       dependencies, architectural context]                │
  │                                                          │
  │      ## Before You Begin                                 │
  │      [If you have questions, ask them now]                │
  │                                                          │
  │      ## Your Job                                         │
  │      1. Implement exactly what the task specifies        │
  │      2. Write tests (following TDD)                      │
  │      3. Verify implementation works                      │
  │      4. Commit your work                                 │
  │      5. Self-review                                      │
  │      6. Report back                                      │
  │                                                          │
  │      Work from: [directory]                              │
  │                                                          │
  │  五个上下文决策点：                                      │
  │                                                          │
  │  1. "FULL TEXT" — 禁止子智能体读文件                     │
  │     → Controller 是信息中转站，不是文件指针              │
  │     → 子智能体可能找不到文件或读到错误版本              │
  │                                                          │
  │  2. "Scene-setting" — Controller 的翻译工作              │
  │     → 不在计划文件中——Controller 把主会话的             │
  │       理解浓缩为场景设定                                 │
  │     → 例如："解析器在 Task 1 中已创建"                  │
  │                                                          │
  │  3. "Work from: [directory]" — 路径传递                  │
  │     → worktree 场景下目录可能不是主项目目录              │
  │     → Controller 必须传递正确路径                        │
  │                                                          │
  │  4. "Ask them now" — 唯一的信息回传通道                  │
  │     → 子智能体提问 → Controller 中转 → 回答后重新派发   │
  │                                                          │
  │  5. 报告格式 — 四种状态的标准化输出                      │
  │     → DONE / DONE_WITH_CONCERNS / BLOCKED / NEEDS_CTX    │
  │     → 自审清单：完整性、质量、纪律、测试                │
  │     → "When You're in Over Your Head" 升级机制           │
  └──────────────────────────────────────────────────────────┘
```

## 规范审阅者子智能体的上下文构造

```
  ┌──────────────────────────────────────────────────────────┐
  │  Controller 派发规范审阅者时的 Task prompt 结构：         │
  │                                                          │
  │  Task tool (general-purpose):                            │
  │    description: "Review spec compliance for Task N"       │
  │    prompt: |                                             │
  │      You are reviewing whether an implementation         │
  │      matches its specification.                          │
  │                                                          │
  │      ## What Was Requested                               │
  │      [FULL TEXT of task requirements]                    │
  │                                                          │
  │      ## What Implementer Claims They Built               │
  │      [From implementer's report]                         │
  │                                                          │
  │      ## CRITICAL: Do Not Trust the Report                │
  │      The implementer finished suspiciously quickly.       │
  │      Their report may be incomplete, inaccurate,          │
  │      or optimistic. You MUST verify everything            │
  │      independently.                                      │
  │                                                          │
  │  上下文构造的五个决策：                                  │
  │                                                          │
  │  1. "What Was Requested" 与实现者看到的任务对齐           │
  │     → 确保两者对"需求是什么"有相同理解                 │
  │     → 如果需求不同，审阅者的判断就不公正               │
  │                                                          │
  │  2. "What Implementer Claims They Built"                  │
  │     → Controller 从实现者报告中提取关键信息              │
  │     → 不是复制全文——而是提取"声称做了什么"             │
  │     → Controller 做了信息过滤和重组                     │
  │                                                          │
  │  3. "Do Not Trust the Report" — 最重要的框架设定          │
  │     → "suspiciously quickly" 是精心设计的措辞             │
  │     → 不是事实陈述——审阅者不知道实现者花了多久          │
  │     → 是预设立场——预设实现者报告不可信                  │
  │     → 类比：审计师查原始凭证，不只看财务报表            │
  │                                                          │
  │  4. 审阅者不看到场景设定                                 │
  │     → 场景设定是给实现者的"辅助理解"                    │
  │     → 审阅者不需要"理解为什么这样设计"                  │
  │     → 场景设定可能引入偏见——审阅者可能因"理解"而放松   │
  │                                                          │
  │  5. 审阅者只做三件事                                     │
  │     → Missing requirements（漏了什么）                   │
  │     → Extra/unneeded work（多了什么）                    │
  │     → Misunderstandings（理解错了什么）                  │
  │     → 三个维度覆盖规范偏离的所有可能                    │
  │     → 不评价代码质量（那是代码审阅者的事）              │
  └──────────────────────────────────────────────────────────┘
```

## 代码审阅者子智能体的上下文构造

```
  ┌──────────────────────────────────────────────────────────┐
  │  Controller 派发代码审阅者时的 Task 参数：                │
  │                                                          │
  │  Task tool (superpowers:code-reviewer):                  │
  │    WHAT_WAS_IMPLEMENTED: [from implementer's report]     │
  │    PLAN_OR_REQUIREMENTS: Task N from [plan-file]         │
  │    BASE_SHA: [commit before task]                        │
  │    HEAD_SHA: [current commit]                            │
  │    DESCRIPTION: [task summary]                           │
  │                                                          │
  │  上下文构造的关键决策：                                  │
  │                                                          │
  │  1. 使用 code-reviewer 类型而非 general-purpose           │
  │     → 专门的智能体定义（code-reviewer.md 模板）          │
  │     → 不需要从零构造行为规则                             │
  │     → 审阅维度：Strengths / Issues / Assessment          │
  │                                                          │
  │  2. BASE_SHA 和 HEAD_SHA — 代码审阅者独有的上下文        │
  │     → 实现者和规范审阅者都不需要 git SHA               │
  │     → SHA 界定审阅范围：只看本次任务变更                │
  │     → Controller 必须在派发前获取这两个 SHA             │
  │                                                          │
  │  3. 顺序：代码审阅在规范审阅之后                         │
  │     → "Only dispatch after spec compliance               │
  │        review passes."                                   │
  │     → 先保证做对了，再保证做好了                        │
  │     → 如果先代码审阅，可能优化了错误的方向              │
  │                                                          │
  │  4. 额外的质量检查维度（Controller 附加）                │
  │     → 文件职责是否清晰                                  │
  │     → 单元是否可独立理解和测试                          │
  │     → 是否遵循计划的文件结构                            │
  │     → 是否创建了过大的新文件                            │
  │     → 这些是 subagent-driven 的工作流需求               │
  │       不是 code-reviewer.md 原有的                      │
  │                                                          │
  │  三种子智能体的信息量对比：                              │
  │  实现者：任务全文 + 场景设定 + 行为规则                  │
  │  规范审阅者：需求全文 + 实现者报告 + 不信任框架          │
  │  代码审阅者：git diff + 任务摘要 + 审阅模板              │
  │  → 信息量相当，但内容完全不同                           │
  │  → 没有一个子智能体拥有"全貌"                          │
  └──────────────────────────────────────────────────────────┘
```
