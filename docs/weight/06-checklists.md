# 06 清单结构

清单（Checklist）是结构性权重中**最可追踪**的类型——它将流程步骤外部化为可勾选项，并通过 TodoWrite 工具强制 AI 逐项完成。

## 3 套核心清单

### 6.1 头脑风暴清单（9 项）

**来源**：`skills/brainstorming/SKILL.md:20-32`

```
You MUST create a task for each of these items and complete them in order:

1. Explore project context — check files, docs, recent commits
2. Offer visual companion (if visual questions ahead)
3. Ask clarifying questions — one at a time
4. Propose 2-3 approaches — with trade-offs and recommendation
5. Present design — in sections, get approval after each
6. Write design doc — save to docs/superpowers/specs/
7. Spec self-review — placeholders, contradictions, ambiguity
8. User reviews written spec
9. Transition to implementation — invoke writing-plans skill
```

**结构特征**：

| 维度 | 分析 |
|------|------|
| "MUST create a task" | 清单不只是列表，必须转化为 TodoWrite 任务 |
| "complete them in order" | 顺序不可打乱 |
| 项 9 = 唯一出口 | 终态锁定为 writing-plans，不能跳到其他技能 |
| 项 2 有条件 | "if visual questions ahead"——条件分支嵌入清单 |

### 6.2 TDD 验证清单（8 项）

**来源**：`skills/test-driven-development/SKILL.md:328-338`

```
Before marking work complete:

- [ ] Every new function/method has a test
- [ ] Watched each test fail before implementing
- [ ] Each test failed for expected reason (feature missing, not typo)
- [ ] Wrote minimal code to pass each test
- [ ] All tests pass
- [ ] Output pristine (no errors, warnings)
- [ ] Tests use real code (mocks only if unavoidable)
- [ ] Edge cases and errors covered

Can't check all boxes? You skipped TDD. Start over.
```

**结构特征**：

| 维度 | 分析 |
|------|------|
| `- [ ]` 复选框语法 | Markdown checkbox——AI 逐项确认 |
| "Can't check all boxes?" | 部分完成 = 未完成——没有"差不多" |
| "Start over" | 不允许补充——必须从头重来 |
| 项 2-3 验证 "失败" | 不只验证"通过"，还验证"失败是否正确" |

### 6.3 写计划自审清单（3 项）

**来源**：`skills/writing-plans/SKILL.md:122-132`

```
After writing the complete plan:

1. Spec coverage: Skim each section. Can you point to a task that implements it?
2. Placeholder scan: Search for TBD, TODO, "implement later", vague requirements
3. Type consistency: Do types/method signatures match between tasks?

If you find issues, fix them inline. If you find a spec requirement with no task, add the task.
```

**结构特征**：

| 维度 | 分析 |
|------|------|
| "after writing" | 时序约束——完成后才能审查 |
| "fix them inline" | 修复方式指定——就地修改，不另建文档 |
| "add the task" | 缺口处理——发现遗漏不是"记录"，是"补上" |

## Checklist → TodoWrite：外部可追踪

```
  ┌──────────────────────────────────────────────────────────┐
  │  清单的权重来源不在于"列出步骤"，                          │
  │  而在于"将步骤外部化为可追踪任务"                          │
  │                                                          │
  │  普通列表：                                               │
  │  1. Explore context                                      │
  │  2. Ask questions                                        │
  │  3. Present design                                       │
  │  → AI 可能在心里"标记完成"而不实际执行                    │
  │                                                          │
  │  TodoWrite 任务：                                        │
  │  [pending] 1. Explore project context                    │
  │  [in_progress] 2. Ask clarifying questions               │
  │  [completed] 3. Present design                           │
  │  → 状态是外部可见的——用户能看到 AI 跳过了什么            │
  │  → 状态转换有约束——必须按顺序，不能跳步                   │
  │  → 不完成 = 用户可见的 "pending" 项                       │
  └──────────────────────────────────────────────────────────┘
```

## 清单的完整性验证

TDD 清单的最后一行体现了清单权重的核心机制：

```
  "Can't check all boxes? You skipped TDD. Start over."

  这不是"建议你重做"——这是"定义了你没有完成 TDD"。

  清单 = 完成条件的精确定义：
  - 没有清单："TDD 做了" → AI 可以自己判断什么算"做了"
  - 有清单：8 项全勾 = TDD 做了，否则 = 没做

  清单消除了"差不多完成了"的灰色地带。
```

## 清单 vs 流程图的互补

```
  清单和流程图控制同一个流程的不同维度：

  流程图控制：
  ──────────
  - 步骤之间的顺序（先 A 后 B）
  - 分支条件（yes → 左，no → 右）
  - 回环路径（不通过 → 回到修复）

  清单控制：
  ────────
  - 步骤本身的完成度（每一项是否做了）
  - 验证条件（测试是否真的看了失败）
  - 终态定义（8 项全勾 = 完成，否则 = 未完成）

  组合效果：
  - 流程图：你不能跳过步骤
  - 清单：你不能敷衍步骤
  - 两者叠加：必须按顺序做，且每步必须做到位
```

## 清单项的粒度控制

```
  头脑风暴清单（粒度较粗）：
  ──────────────────────────
  1. Explore project context
  2. Offer visual companion
  3. Ask clarifying questions
  → 每项是"一个活动"，不是"一个动作"
  → AI 有裁量空间决定"探索"到什么程度

  TDD 清单（粒度极细）：
  ────────────────────────
  - [ ] Every new function/method has a test
  - [ ] Watched each test fail before implementing
  - [ ] Each test failed for expected reason
  → 每项是"一个可验证的条件"
  → AI 没有裁量空间——要么看了失败，要么没看

  写计划清单（粒度中等）：
  ────────────────────────
  1. Spec coverage
  2. Placeholder scan
  3. Type consistency
  → 每项是"一个审查维度"
  → AI 需要实际做审查，但审查深度有裁量空间

  粒度选择原则：
  - 需要精确控制 → 细粒度（TDD）
  - 需要灵活执行 → 粗粒度（头脑风暴）
  - 需要确保质量 → 中粒度（自审）
```

## "MUST create a task" 的双重强制

```
  头脑风暴清单的指令：

  "You MUST create a task for each of these items
   and complete them in order"

  双重强制：
  ────────
  1. "create a task" → 必须转化为 TodoWrite（外部追踪）
  2. "complete them in order" → 必须按顺序完成（顺序锁定）

  如果只说"complete them"：
  → AI 可能跳步、可能并行、可能只做一部分

  "create a task" + "in order"：
  → TodoWrite 中的 pending/in_progress/completed 状态可见
  → 顺序约束让 AI 不能先做第 5 项再做第 3 项
  → 用户能实时看到进度和跳步
```
