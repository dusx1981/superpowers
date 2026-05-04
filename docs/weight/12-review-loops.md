# 12 审阅循环

审阅循环是结构性权重中**最强制纠偏**的类型——审阅不通过不是"建议改进"，而是"必须修复 + 重新审阅"，不能跳过。

## 审阅循环的架构

```
  ┌──────────────────────────────────────────────────────────┐
  │  两阶段审阅 + 回环机制：                                │
  │                                                          │
  │  实现者完成                                              │
  │      │                                                   │
  │      ▼                                                   │
  │  ┌─────────────────┐                                    │
  │  │ 规范审阅者       │                                    │
  │  │ "代码符合规范吗？"│                                    │
  │  └────────┬────────┘                                    │
  │           │                                              │
  │     ┌─────┴─────┐                                       │
  │     │           │                                       │
  │    不通过       通过                                      │
  │     │           │                                       │
  │     ▼           ▼                                       │
  │  实现者修复   ┌─────────────────┐                       │
  │     │        │ 代码质量审阅者    │                       │
  │     │        │ "代码质量好吗？"  │                       │
  │     │        └────────┬────────┘                        │
  │     │              ┌───┴───┐                             │
  │     │              │       │                             │
  │     │            不通过   通过                            │
  │     │              │       │                             │
  │     │              ▼       ▼                             │
  │     │           实现者修复  标记任务完成                  │
  │     │              │                                    │
  │     └──→ 重新规范审阅 ←─┘                               │
  │                                                          │
  │  关键：不通过 → 修复 → 重新审阅（不是"建议改进就完事"）│
  └──────────────────────────────────────────────────────────┘
```

## 审阅循环的来源

**来源**：`skills/subagent-driven-development/SKILL.md:234-259`

```
**Never:**
- Skip reviews (spec compliance OR code quality)
- Proceed with unfixed issues
- Accept "close enough" on spec compliance
- Skip review loops (reviewer found issues = implementer
  fixes = review again)
- Let implementer self-review replace actual review
- Start code quality review before spec compliance is ✅
- Move to next task while either review has open issues

**If reviewer finds issues:**
- Implementer (same subagent) fixes them
- Reviewer reviews again
- Repeat until approved
- Don't skip the re-review
```

## 为什么审阅必须循环？

```
  ┌──────────────────────────────────────────────────────────┐
  │  一次审阅 vs 循环审阅：                                 │
  │                                                          │
  │  一次审阅（弱）：                                       │
  │  审阅者发现问题 → 实现者修复 → 结束                     │
  │  问题：修复可能引入新问题，但没人检查                   │
  │                                                          │
  │  循环审阅（强）：                                       │
  │  审阅者发现问题 → 实现者修复 → 审阅者重新审阅           │
  │  → 可能发现新问题 → 实现者修复 → 审阅者重新审阅         │
  │  → 直到审阅者批准                                       │
  │                                                          │
  │  循环保证了：                                            │
  │  1. 修复确实生效了                                      │
  │  2. 修复没有引入新问题                                  │
  │  3. 最终状态是审阅者确认的，不是实现者声称的            │
  └──────────────────────────────────────────────────────────┘
```

## 审阅顺序锁

```
  ┌──────────────────────────────────────────────────────────┐
  │  两阶段审阅有严格的顺序：                               │
  │                                                          │
  │  Phase 1: 规范审阅 → 确认代码符合规范                   │
  │  Phase 2: 代码质量审阅 → 确认代码质量好                 │
  │                                                          │
  │  顺序不可颠倒的原因：                                   │
  │  - 规范审阅检查"做了对的事"                            │
  │  - 代码质量审阅检查"把事做对了"                        │
  │  - 如果做了错的事（规范不符），质量再好也要改            │
  │  - 先做质量审阅可能发现的问题在规范审阅后就不存在了     │
  │                                                          │
  │  Red Flags 明确禁止：                                    │
  │  "Never start code quality review before                 │
  │   spec compliance is ✅"                                 │
  │                                                          │
  │  顺序锁防止的常见错误：                                 │
  │  - 代码写得很漂亮但不符合规范                           │
  │  - 质量审阅通过了但功能缺失                             │
  │  - 审阅者浪费时间评价不该存在的代码                     │
  └──────────────────────────────────────────────────────────┘
```

## "Accept close enough" 的禁止

```
  规范审阅的二元判断：

  ✅ Spec compliant — all requirements met, nothing extra
  ❌ Issues found: [list specifically what's missing or extra,
                    with file:line references]

  "Accept 'close enough' on spec compliance" 被明确禁止。

  为什么？

  - "close enough" 意味着有些需求没满足
  - 规范审阅者的职责是精确比对
  - ✅ 和 ❌ 之间没有 "差不多 ✅"
  - 漏一个需求 = ❌，不是 "✅ 但有小问题"

  这和结构化状态（08-structured-status.md）的原理相同：
  - 二值判断消除了灰色地带
  - "close enough" 是灰色地带的温床
  - 明确禁止 "close enough" = 关闭灰色地带
```

## 自审 ≠ 审阅

```
  ┌──────────────────────────────────────────────────────────┐
  │  实现者有自审环节，但自审不能替代正式审阅：             │
  │                                                          │
  │  实现者自审（Prompt 模板中的 Self-Review）：            │
  │  - Completeness: 是否实现了所有需求？                    │
  │  - Quality: 是否是最好的工作？                           │
  │  - Discipline: 是否遵循了 YAGNI？                       │
  │  - Testing: 测试是否真正验证了行为？                    │
  │                                                          │
  │  自审的作用：                                            │
  │  - 在交出代码前发现明显问题                              │
  │  - 提高首次提交的质量                                    │
  │  - 减少审阅循环的次数                                    │
  │                                                          │
  │  自审不能替代审阅的原因：                                │
  │  - 确认偏误：自己写的代码自己容易"觉得没问题"           │
  │  - 视角局限：实现者知道自己做了什么，                    │
  │    不知道自己应该做什么（规范）                          │
  │  - 利益冲突：实现者想"通过"，审阅者想"确保正确"        │
  │                                                          │
  │  Red Flags 明确禁止：                                    │
  │  "Never let implementer self-review                      │
  │   replace actual review (both are needed)"               │
  └──────────────────────────────────────────────────────────┘
```

## 审阅循环的终止条件

```
  循环何时终止？

  唯一终止条件：审阅者返回 ✅/Approved

  非终止条件：
  - 实现者说"修好了" → 不是终止条件，必须审阅者确认
  - 审阅者说"问题不大" → 不是终止条件，必须 Approved
  - 修了 3 轮还是不通过 → 不是终止条件，继续修

  没有超时机制：
  - 审阅循环没有"最多 N 轮"的限制
  - 没通过就是没通过，必须继续
  - 这保证了最终质量，而非"差不多就过"

  实际的退出路径：
  - 审阅者批准 → 正常退出
  - 实现者 BLOCKED → 升级给 Controller
  - Controller 决定换模型/拆任务/升级人类
  - 这是"架构层面"的退出，不是"审阅层面"的退出
```
