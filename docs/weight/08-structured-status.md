# 08 结构化状态

结构化状态是结构性权重中**最精确**的类型——用有限的状态码替代自由文本，消除 AI 在报告中的模糊空间。

## 四种状态码

**来源**：`skills/subagent-driven-development/SKILL.md:104-117`、`skills/subagent-driven-development/implementer-prompt.md:103-112`

```
DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
```

### 状态定义

| 状态码 | 含义 | Controller 行动 |
|--------|------|----------------|
| `DONE` | 任务完成，无需额外处理 | 进入规范审阅 |
| `DONE_WITH_CONCERNS` | 任务完成但有问题虑 | 先读问题虑，再决定是否进入审阅 |
| `BLOCKED` | 无法完成任务 | 评估阻塞：加上下文/换模型/拆任务/升级人类 |
| `NEEDS_CONTEXT` | 缺少必要信息 | 补充信息后重新调度 |

### 为什么不用自由文本？

```
  自由文本报告：
  ──────────────
  "I've mostly finished the implementation. There are a few
   things I'm not sure about, but I think it should work.
   I might need more information about the database schema."

  AI 解读空间：
  - "mostly finished" → 完成度 70%? 90%?
  - "a few things" → 2 件? 5 件? 严重吗?
  - "should work" → 确信? 猜测?
  - "might need" → 需要还是不需要?

  结构化状态报告：
  ──────────────────
  "Status: DONE_WITH_CONCERNS
   - Implemented retry logic with exponential backoff
   - Tests: 8/8 passing
   - Concern: retry interval config not specified, used 1s default
   - Files: src/retry.ts, tests/retry.test.ts"

  AI 解读空间：
  - DONE_WITH_CONCERNS → 完成了但有疑虑，必须先读疑虑
  - 具体疑虑一目了然
  - 没有模糊措辞
```

## 状态转换图

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  调度实现者                                              │
  │      │                                                   │
  │      ▼                                                   │
  │  ┌──────────────┐                                       │
  │  │ NEEDS_CONTEXT │──→ 补充信息 ──→ 重新调度              │
  │  └──────────────┘                                       │
  │      │                                                   │
  │      ▼ (不是 NEEDS_CONTEXT)                              │
  │  ┌──────────────┐                                       │
  │  │   BLOCKED     │──→ 评估阻塞 ──→ 加上下文/换模型/     │
  │  └──────────────┘       拆任务/升级人类                  │
  │      │                                                   │
  │      ▼ (不是 BLOCKED)                                    │
  │  ┌─────────────────────┐                                │
  │  │ DONE_WITH_CONCERNS  │──→ 读疑虑 ──→ 决定是否进审阅   │
  │  └─────────────────────┘                                │
  │      │                                                   │
  │      ▼ (不是 DONE_WITH_CONCERNS)                        │
  │  ┌──────┐                                               │
  │  │ DONE │──→ 进入规范审阅                                │
  │  └──────┘                                               │
  │                                                          │
  └──────────────────────────────────────────────────────────┘

  关键：状态码是排他的——AI 必须选择一个，不能选两个
  不能说"DONE 但也 NEEDS_CONTEXT"——必须二选一
```

## BLOCKED 状态的升级机制

```
  BLOCKED 不是失败——是信号。

  Controller 收到 BLOCKED 后的决策树：
  ──────────────────────────────

  1. 是上下文问题？
     → 提供更多上下文，用相同模型重新调度

  2. 是能力问题？
     → 用更强模型重新调度

  3. 任务太大？
     → 拆分为更小的任务

  4. 计划本身有误？
     → 升级给人类

  关键约束：
  "Never ignore an escalation or force the same model
   to retry without changes."

  如果实现者说卡住了 → 必须改变某些条件再重试
  不能什么都不改就重新调度同一个模型
```

## "Never silently produce work you're unsure about"

```
  ┌──────────────────────────────────────────────────────────┐
  │  实现者提示词中的关键指令：                              │
  │                                                          │
  │  "Use DONE_WITH_CONCERNS if you completed the work       │
  │   but have doubts about correctness.                     │
  │   Use BLOCKED if you cannot complete the task.           │
  │   Use NEEDS_CONTEXT if you need information              │
  │   that wasn't provided.                                  │
  │   Never silently produce work you're unsure about."      │
  │                                                          │
  │  这条指令的核心：                                        │
  │  - 不确定 = 必须报告，不是默默提交                      │
  │  - 三种非 DONE 状态覆盖了所有不确定场景                  │
  │  - "silently" 是关键词——不报告 = 更严重的违规           │
  │  - 状态码让"不确定"变得可操作——不是模糊描述             │
  │    而是明确信号                                           │
  └──────────────────────────────────────────────────────────┘
```

## 审阅者的结构化输出

```
  规范审阅者输出：
  ──────────────
  - ✅ Spec compliant (if everything matches)
  - ❌ Issues found: [list with file:line references]

  代码质量审阅者输出：
  ──────────────────
  - Strengths: [list]
  - Issues:
    - Critical: [list]
    - Important: [list]
    - Minor: [list]
  - Assessment: Approved | Changes Requested

  结构化特征：
  - ✅/❌ 符号 = 二值判断（通过/不通过）
  - Critical/Important/Minor = 三级严重度
  - file:line = 精确定位
  - Assessment = 最终结论（Approved/Changes Requested）

  为什么不用自由文本？
  - "I think it looks pretty good overall"
    → AI 倾向于积极评价，问题被稀释
  - "Approved"
    → 二值判断，没有稀释空间
```

## 状态码 vs 自由文本的权重对比

```
  ┌──────────────────────────────────────────────────────────┐
  │  同一个意思，状态码 vs 自由文本：                        │
  │                                                          │
  │  自由文本：                                              │
  │  "I think I'm done but I'm not 100% sure about           │
  │   the edge cases. Also I couldn't figure out             │
  │   the database part."                                    │
  │  → Controller 需要解读——"done" 是 DONE 还是             │
  │    DONE_WITH_CONCERNS？"couldn't figure out"             │
  │    是 BLOCKED 还是 NEEDS_CONTEXT？                       │
  │                                                          │
  │  状态码：                                                │
  │  "Status: DONE_WITH_CONCERNS                             │
  │   Concern: edge cases not verified                       │
  │   Blocking: database schema needed"                      │
  │  → Controller 无需解读——DONE_WITH_CONCERNS               │
  │    明确告诉 Controller 先读疑虑再决定                    │
  │                                                          │
  │  权重差异：                                              │
  │  - 自由文本：AI 可以模糊处理，Controller 可能误读        │
  │  - 状态码：AI 必须精确选择，Controller 自动正确处理      │
  │  - 状态码消除了"表达"环节的模糊性                       │
  └──────────────────────────────────────────────────────────┘
```
