# 07 反模式列表

反模式列表是结构性权重中**最具预防性**的类型——不告诉 AI 做什么，而是告诉 AI 什么 = 失败。负面清单比正面指令更难绕过。

## 5 套反模式列表

### 7.1 写计划占位符反模式（6 项）

**来源**：`skills/writing-plans/SKILL.md:108-114`

```
These are plan failures — never write them:

- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — engineer may read out of order)
- Steps that describe what to do without showing how (code blocks required)
- References to types, functions, or methods not defined in any task
```

**权重机制**：

| 维度 | 分析 |
|------|------|
| "plan failures" | 不是"不推荐"，是"失败"——二值判断 |
| "never write them" | 绝对否定——没有"偶尔可以" |
| 每项有具体模式 | 不是泛泛说"不要模糊"，而是列出"TODO"/"TBD"等具体字符串 |
| 有解释 | 每项说明为什么——"engineer may read out of order" |

### 7.2 头脑风暴 "太简单" 反模式

**来源**：`skills/brainstorming/SKILL.md:16-18`

```
Anti-Pattern: "This Is Too Simple To Need A Design"

Every project goes through this process. A todo list,
a single-function utility, a config change — all of them.
"Simple" projects are where unexamined assumptions
cause the most wasted work.
```

**权重机制**：

| 维度 | 分析 |
|------|------|
| "Anti-Pattern" 标题 | 明确标注为反模式——不是"建议"而是"反面教材" |
| 引号模拟内心独白 | "This Is Too Simple..."——AI 的潜在想法被预先否定 |
| 举例穷举 | "todo list, utility, config change"——堵住"但这次真的简单" |
| 因果反转 | "简单项目 = 未审视假设 = 最大浪费"——反直觉但精确 |

### 7.3 TDD 红旗列表（13 项）

**来源**：`skills/test-driven-development/SKILL.md:272-287`

```
Red Flags - STOP and Start Over:

- Code before test
- Test after implementation
- Test passes immediately
- Can't explain why test failed
- Tests added "later"
- Rationalizing "just this once"
- "I already manually tested it"
- "Tests after achieve the same purpose"
- "It's about spirit not ritual"
- "Keep as reference" or "adapt existing code"
- "Already spent X hours, deleting is wasteful"
- "TDD is dogmatic, I'm being pragmatic"
- "This is different because..."

All of these mean: Delete code. Start over with TDD.
```

**权重机制**：

| 维度 | 分析 |
|------|------|
| "STOP and Start Over" | 不是"注意"而是"停下"——动作指令 |
| 行为 + 合理化 | 前 4 项是行为（code before test），后 9 项是合理化 |
| "All of these mean" | 统一结论——不管哪种红旗，处理方式相同 |
| "Delete code" | 极端措施——不是"修正"，是"删除重来" |

### 7.4 验证红旗列表（7 项）

**来源**：`skills/verification-before-completion/SKILL.md:53-61`

```
Red Flags - STOP:

- Using "should", "probably", "seems to"
- Expressing satisfaction before verification
- About to commit/push/PR without verification
- Trusting agent success reports
- Relying on partial verification
- Thinking "just this once"
- Tired and wanting work over
- ANY wording implying success without having run verification
```

**权重机制**：

| 维度 | 分析 |
|------|------|
| 词汇级红旗 | "should"/"probably"/"seems"——精确到用词 |
| 情绪级红旗 | "expressing satisfaction"——精确到情绪状态 |
| 状态级红旗 | "tired and wanting work over"——精确到心理状态 |
| 全称量词 | "ANY wording"——消灭措辞变通的余地 |

### 7.5 调试红旗列表（10 项）

**来源**：`skills/systematic-debugging/SKILL.md:217-229`

```
If you catch yourself thinking:

- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- "Skip the test, I'll manually verify"
- "It's probably X, let me fix that"
- "I don't fully understand but this might work"
- "Pattern says X but I'll adapt it differently"
- "Here are the main problems: [lists fixes without investigation]"
- Proposing solutions before tracing data flow
- "One more fix attempt" (when already tried 2+)

ALL of these mean: STOP. Return to Phase 1.
```

**权重机制**：

| 维度 | 分析 |
|------|------|
| "catch yourself thinking" | 内省式——AI 需要监视自己的思维过程 |
| 引号包裹 | 每条红旗都是内心独白——AI 可以直接匹配 |
| 升级机制 | 第 10 项有条件（"already tried 2+"）→ 质疑架构 |
| 统一结论 | "ALL" → "Return to Phase 1"——不管哪种，都回根因调查 |

## 反模式 vs 正面指令的权重差异

```
  正面指令（弱）：
  ───────────────
  "Write complete code in every step"
  → AI 知道应该做什么，但"差不多完整"也觉得自己做了

  反模式（强）：
  ───────────────
  "These are plan failures:
   - TBD, TODO, implement later
   - Add appropriate error handling
   - Write tests for the above (without code)"

  → AI 知道什么 = 失败
  → 写了 "TODO" = 失败，不是"不太理想"
  → 写了 "handle edge cases" = 失败，不是"可以改进"

  差异：
  - 正面指令定义"成功是什么样的" → AI 可以重新解读
  - 反模式定义"失败是什么样的" → AI 很难重新解读
  - "失败"是二值的——你要么失败了，要么没有
  - "成功"是连续的——"差不多成功"也是"成功"
```

## 反模式的累积效应

```
  ┌──────────────────────────────────────────────────────────┐
  │  反模式不是孤立的——它们形成累积效应                      │
  │                                                          │
  │  TDD 技能中的三层防御：                                  │
  │                                                          │
  │  Layer 1: Iron Law                                       │
  │  "NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST"       │
  │  → 定义铁律                                              │
  │                                                          │
  │  Layer 2: Rationalization Table (12 项)                  │
  │  → 预判合理化想法                                        │
  │                                                          │
  │  Layer 3: Red Flags List (13 项)                         │
  │  → 识别违反行为                                          │
  │                                                          │
  │  三层叠加：                                              │
  │  - 铁律告诉 AI 规则是什么                                │
  │  - 合理化表格告诉 AI "你可能这样想，但不对"             │
  │  - 红旗列表告诉 AI "如果你在这样做，停下"               │
  │                                                          │
  │  即使 AI 绕过了铁律（"这次例外"），                      │
  │  合理化表格会拦截（"太简单不需要测试？"），              │
  │  即使绕过了合理化表格，                                  │
  │  红旗列表会拦截（"测试立刻通过了？"）。                  │
  └──────────────────────────────────────────────────────────┘
```

## 反模式的递进严重度

```
  不同技能的反模式有不同严重度：

  写计划反模式：
  "plan failures" → 失败 = 不可发布，但可以修改后重新提交

  TDD 红旗：
  "Delete code. Start over." → 删除重来 = 之前的工作作废

  调试红旗：
  "STOP. Return to Phase 1." → 停下回退 = 当前进度回滚

  验证红旗：
  "dishonesty, not efficiency" → 道德否定 = 违反 = 不诚实

  严重度递进：
  修改 → 删除 → 回退 → 道德否定
  越往后，AI 越难绕过
```
