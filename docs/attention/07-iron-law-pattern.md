# 07 铁律模式：最高注意力权重的结构设计

铁律是 Superpowers 中注意力权重最高的结构模式。4 个技能使用了铁律，它们共享相同的结构设计但有不同的内容。

## 铁律的结构解剖

```
  ┌──────────────────────────────────────────────────────────┐
  │  铁律的标准结构（4 个技能通用）：                        │
  │                                                          │
  │  Layer 1: "Violating the letter..." 预防声明              │
  │  ────────────────────────────────────────                │
  │  → 位置：技能文件的第 14 行（几乎所有铁律技能一致）      │
  │  → 内容："Violating the letter of [X] is violating        │
  │     the spirit of [X]."                                   │
  │  → 功能：预防"精神 vs 字面"的理性化                     │
  │  → 场景：AI 想"这个规则的精神是 X，我不需要遵守字面"   │
  │  → 反驳：违反字面 = 违反精神 = 没有灰色地带             │
  │                                                          │
  │  Layer 2: Iron Law 主体                                   │
  │  ────────────────────────                                │
  │  → 格式：代码块 + 全大写                                 │
  │  → 句式："NO [X] WITHOUT [Y]"                             │
  │  → 功能：绝对禁止的声明                                  │
  │  → 全大写在训练数据中与法律/警告/危险信号关联            │
  │  → 代码块赋予"不可修改"的语义                           │
  │                                                          │
  │  Layer 3: 即时后果                                       │
  │  ────────────────                                        │
  │  TDD: "Write code before the test? Delete it.              │
  │        Start over."                                       │
  │  debugging: "If you haven't completed Phase 1,             │
  │            you cannot propose fixes."                      │
  │  verification: "If you haven't run the verification        │
  │              command in this message, you cannot           │
  │              claim it passes."                             │
  │  → 功能：不只说"不行"——还说"违反后怎么办"              │
  │  → 后果是即时的、严重的（删除/重做/禁止）               │
  │  → 即时后果提高了违反的心理成本                           │
  │                                                          │
  │  Layer 4: "No exceptions" 清单                            │
  │  ─────────────────────────                                │
  │  TDD: 4 条例外清单                                       │
  │    - "Don't keep it as 'reference'"                       │
  │    - "Don't 'adapt' it while writing tests"               │
  │    - "Don't look at it"                                   │
  │    - "Delete means delete"                                │
  │  → 功能：逐条关闭 AI 可能想到的"例外"                   │
  │  → 每一条都是 AI 理性化的常见路径                       │
  │  → 关闭了所有"但是..."的入口                            │
  │                                                          │
  │  Layer 5: Final Rule（重述）                              │
  │  ─────────────────────────                                │
  │  TDD: "Production code → test exists and failed first"     │
  │  → 位置：技能文件的最末尾                                │
  │  → 功能：利用近因效应，让 AI 带着约束离开              │
  │  → "Otherwise → not TDD" = 二元判断                     │
  │  → 没有中间状态——要么是 TDD，要么不是                  │
  │                                                          │
  │  五层结构的注意力权重分布：                              │
  │  Layer 1 (预防声明): 7                                   │
  │  Layer 2 (Iron Law 主体): 10                              │
  │  Layer 3 (即时后果): 8                                   │
  │  Layer 4 (No exceptions): 7                               │
  │  Layer 5 (Final Rule): 9                                  │
  │  → 主体最强（全大写+代码块+XML级信号）                  │
  │  → 末尾次强（近因效应+重述主体）                       │
  │  → 预防和例外清单中等（补充性约束）                     │
  │  → 后果陈述中强（提供了违反的心理成本）                 │
  └──────────────────────────────────────────────────────────┘
```

## 铁律的"NO X WITHOUT Y"句式分析

```
  ┌──────────────────────────────────────────────────────────┐
  │  为什么 "NO X WITHOUT Y" 比简单的 "DO Y" 更有效？       │
  │                                                          │
  │  对比：                                                  │
  │  弱版本："Always write a failing test before code"         │
  │  → 祈使句式 = 建议/命令                                  │
  │  → AI 可能认为"这次不需要"是合理的例外                  │
  │  → 裁量空间大——AI 可以判断"什么时候需要"               │
  │                                                          │
  │  强版本："NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST" │
  │  → 条件句式 = 逻辑上的不可能                            │
  │  → 不是"应该写测试"——而是"没有测试就不能写代码"        │
  │  → 裁量空间为零——不存在"没有测试也能写代码"的情况      │
  │  → 从"建议"变成"前提条件"                              │
  │                                                          │
  │  句式的逻辑结构：                                        │
  │  "NO X WITHOUT Y" = X → Y（X 蕴含 Y）                   │
  │  → 逻辑等价：如果你做了 X，那么 Y 必须已经完成          │
  │  → 这是比 "DO Y" 更强的约束                              │
  │  → 因为它不只要求 Y——它要求 Y 在 X 之前                │
  │  → 时序约束比存在约束更强                               │
  │                                                          │
  │  四个铁律的句式统一性：                                  │
  │  TDD: NO CODE WITHOUT TEST FIRST                          │
  │  Debug: NO FIXES WITHOUT INVESTIGATION FIRST              │
  │  Verify: NO CLAIMS WITHOUT EVIDENCE FIRST                 │
  │  Write: NO SKILL WITHOUT TEST FIRST                       │
  │  → 全部是 "NO [行动] WITHOUT [前提] FIRST"               │
  │  → 统一格式 = 可识别的模式                               │
  │  → AI 在不同技能中看到相同格式 → 理解"这是铁律"        │
  │  → 模式识别增强了服从概率                                │
  └──────────────────────────────────────────────────────────┘
```
