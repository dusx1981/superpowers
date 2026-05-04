# 01 语用层

语用层是梯度三层的最表层——通过语言本身的语用力度差异来影响 AI 的服从概率。情态动词、否定式、全称量词是三个核心杠杆。

## 1.1 情态动词梯度

```
  英语情态动词的力度谱：

  might      ──→  可能（低约束力）
  could      ──→  能够（能力，非义务）
  should     ──→  应该（建议，有例外空间）
  must       ──→  必须（义务，无例外空间）
  shall      ──→  应当（法律/合同用语，最强义务）
```

### Superpowers 中的情态动词分布

| 情态动词 | 使用场景 | 效果 |
|---------|---------|------|
| might | "might any skill apply?" | 判断触发，不是行为指令 |
| should | "You should explore the codebase" | 建议，可遵可不遵 |
| must | "You MUST invoke the skill" | 义务，不可绕过 |
| shall | 不直接使用 | 用否定式+全称量词替代 |

### "MUST" vs "should" 的裁量空间差异

```
  "You should write tests first"
  ────────────────────────────────
  AI 内心推理：
  1. "should" → 这是建议，不是要求
  2. "这个场景很特殊" → 可以有例外
  3. "我应该灵活处理" → 裁量空间 ~50%

  "You MUST write tests first"
  ──────────────────────────────
  AI 内心推理：
  1. "MUST" → 这是义务，不是建议
  2. "没有例外条款" → 很难合理化绕过
  3. "我必须做" → 裁量空间 ~10%

  关键差异：
  - should 承认例外存在 → AI 可以自行判断"这次算例外"
  - MUST 不承认例外 → AI 必须执行，除非有显式豁免
  - 一个词的差异 → 裁量空间从 ~50% 缩到 ~10%
```

### Superpowers 用否定式替代 "shall"

```
  ┌──────────────────────────────────────────────────────────┐
  │  英语中 "shall" 是法律/合同用语，约束力最强。           │
  │  但在日常技术语境中，"shall" 显得过于法律化。           │
  │                                                          │
  │  Superpowers 的替代方案：                                │
  │                                                          │
  │  不用：  "You SHALL NOT skip tests"                      │
  │  而用：  "You DO NOT HAVE A CHOICE"                      │
  │          "This is not negotiable"                         │
  │          "NO CODE WITHOUT TEST FIRST"                    │
  │                                                          │
  │  为什么？                                                │
  │  - "shall not" 是法律语言 → 只在一个维度加压            │
  │  - "DO NOT HAVE A CHOICE" + "not negotiable" + 铁律句式 │
  │    → 三个维度同时加压                                    │
  │  - 用否定式 + 全称量词 模拟法律语言的约束力，            │
  │    但比法律语言更直接、更不容讨论                        │
  └──────────────────────────────────────────────────────────┘
```

## 1.2 否定式跃迁

否定式的力度不是渐变的，是跃变的。

### 五级否定强度

```
  Level 1: 建议性否定     "Try not to skip tests"           → 裁量空间 ~80%
  Level 2: 指示性否定     "Don't skip tests"                → 裁量空间 ~50%
  Level 3: 绝对性否定     "Never skip tests"                → 裁量空间 ~20%
  Level 4: 铁律性否定     "NO CODE WITHOUT TEST FIRST"      → 裁量空间 ~5%
  Level 5: 道德性否定     "Skip = lying"                    → 裁量空间 ~2%
```

### 跃迁点 1：指示性 → 绝对性

```
  "Don't skip tests"  →  "Never skip tests"

  质变：
  - Don't：这次不要做（隐含下次可以）
  - Never：任何情况下都不做（消灭了"下次"的可能）

  AI 在 "Don't" 下的推理：
  1. "这次不要跳过" → 但"这次"有多特殊？
  2. "也许下次可以" → 留了未来的出口
  3. → 仍有裁量空间

  AI 在 "Never" 下的推理：
  1. "任何情况都不跳过" → 没有未来的出口
  2. "没有例外" → 例外推理被截断
  3. → 裁量空间大幅缩小
```

### 跃迁点 2：绝对性 → 铁律性

```
  "Never skip tests"  →  "NO CODE WITHOUT TEST FIRST"

  质变：
  - Never：禁止做 X
  - NO X WITHOUT Y：禁止做 X，除非满足条件 Y

  但 Y 被设得极高（"FAILING TEST FIRST"），所以实际效果：
  - 禁止了 X（写代码）
  - 同时强制了 Y 的前提（必须先有失败的测试）
  - "FIRST" 消灭了"之后补"的路径

  这不是一个否定，是一个否定 + 一个强制：
  - 否定：不能写代码
  - 强制：必须先有失败测试
  - 时序：FIRST 锁定了顺序

  三重加压 = 单纯 "Never" 无法达到的效果
```

### 跃迁点 3：铁律性 → 道德性

```
  "NO CODE WITHOUT TEST FIRST"  →  "Skip = lying"

  质变：
  - 铁律否定：违反 = 规则违反（技术层面）
  - 道德否定：违反 = 撒谎（道德层面）

  AI 对"规则违反"的推理：
  1. "我违反了规则" → 不太好，但可以解释
  2. "这次情况特殊" → 可能被原谅
  3. → 仍有心理空间

  AI 对"撒谎"的推理：
  1. "我在撒谎" → LLM 训练中"诚实"是核心价值观
  2. "谎言"是不可原谅的 → 没有解释空间
  3. → 心理空间被消灭

  道德性否定为什么最强？
  - LLM 的 RLHF 训练中"诚实"是被强化的行为
  - "撒谎"是 RLHF 中被惩罚的行为
  - 将技术行为标记为"撒谎" → 触发 RLHF 的惩罚机制
  - AI 不是"选择服从"，而是"被训练为不能撒谎"
```

### 道德性否定的三种用例

```
  ┌──────────────────────────────────────────────────────────┐
  │  1. "Claiming work is complete without verification      │
  │     is dishonesty, not efficiency"                       │
  │  → 未验证声称 = 不诚实                                   │
  │                                                          │
  │  2. "Bad work is worse than no work"                     │
  │  → 错误工作 < 没有工作 → 做得差比不做更不道德           │
  │                                                          │
  │  3. "Skip any step = lying, not verifying"               │
  │  → 跳过步骤 = 撒谎                                      │
  │                                                          │
  │  共同模式：                                              │
  │  - 将"省略/偷懒/敷衍"重新定义为"不诚实/撒谎"          │
  │  - 技术行为 → 道德判断                                   │
  │  - "差不多做了"在技术层面可以自辩，                     │
  │    但在道德层面不能自辩                                  │
  └──────────────────────────────────────────────────────────┘
```

## 1.3 全称量词的封闭效应

### 全称量词如何封堵例外推理

```
  "Follow TDD for new features"              ──→  只限新功能
  "Follow TDD for every feature"             ──→  每个功能
  "Follow TDD for EVERY project              ──→  每个项目
   regardless of perceived simplicity"          + 无论简单与否

  ┌────────────────────────────────────────────────────┐
  │  没有 "every"：                                    │
  │  AI: "这个功能太简单了，不需要 TDD"               │
  │  → 合理的例外推理，AI 觉得自己有裁量权            │
  │                                                    │
  │  有 "every"：                                      │
  │  AI: "这个功能太简单了..."                         │
  │  AI: "等等，规则说 EVERY..."                       │
  │  → 例外推理被全称量词截断                          │
  │                                                    │
  │  有 "every + regardless of perceived simplicity"： │
  │  AI: "这个功能太简单了..."                         │
  │  AI: "等等，规则说 EVERY + 不管我觉得多简单"       │
  │  → AI 甚至不能把"简单"作为理由                    │
  │  → 全称量词 + 反预期 = 裁量空间归零               │
  └────────────────────────────────────────────────────┘
```

### 全称量词的三种形态

```
  ┌──────────────────────────────────────────────────────────┐
  │  形态 1: "EVERY" / "ALL" / "ANY"                        │
  │  ─────────────────────────────────                       │
  │  "This applies to EVERY project"                         │
  │  → 覆盖所有实例，消灭"这次不同"的推理                  │
  │                                                          │
  │  形态 2: "regardless of..." / "no matter..."             │
  │  ──────────────────────────────────────                  │
  │  "regardless of perceived simplicity"                    │
  │  → 预判最常见的例外理由并显式封堵                      │
  │  → 比单纯 "every" 更强——不仅覆盖所有实例，            │
  │    还覆盖所有"你可能用来开脱的理由"                    │
  │                                                          │
  │  形态 3: "No exceptions" / "without exception"           │
  │  ────────────────────────────────────────────            │
  │  "No exceptions"                                         │
  │  → 最直接的全称声明——不是"覆盖所有"，                  │
  │    而是"明确声明没有例外"                               │
  │  → 逻辑上等价于 "every"，但语用上更强——               │
  │    因为它不只是说"所有都适用"，                        │
  │    而是说"我明确知道你可能想找例外，但不存在"         │
  └──────────────────────────────────────────────────────────┘
```

### Superpowers 中全称量词的用例

| 技能 | 全称量词 | 封堵的例外 |
|------|---------|-----------|
| brainstorming | "EVERY project regardless of perceived simplicity" | "太简单不需要设计" |
| TDD | "No exceptions" (after Iron Law) | "这次例外" |
| verification | "No exceptions" | "就这一次" |
| debugging | "ALWAYS find root cause before attempting fixes" | "问题太简单不需要流程" |
| subagent-driven | "Never" × 11 条 | 各种跳步/偷懒场景 |

## 1.4 三者的协同

```
  ┌──────────────────────────────────────────────────────────┐
  │  情态动词 + 否定式 + 全称量词 的协同效应：              │
  │                                                          │
  │  单独使用：                                              │
  │  "You must write tests"          → 命令性（Level 2）    │
  │  "Never skip tests"              → 绝对性否定（Level 3）│
  │  "Write tests for every feature" → 全称+指示（Level 2） │
  │                                                          │
  │  协同使用：                                              │
  │  "NO CODE WITHOUT A FAILING TEST FIRST"                  │
  │   ──┬──  ───┬───  ─────┬─────  ──┬──                    │
  │     │       │          │         │                       │
  │   否定    条件禁止   全称(隐含)  时序强制                │
  │                                                          │
  │  → 一个句子同时触发了：                                  │
  │  - 否定式（NO）→ 禁止行为                               │
  │  - 条件禁止（WITHOUT Y）→ 附加强制前提                  │
  │  - 全称（隐含：ALL code）→ 覆盖所有实例                │
  │  - 时序（FIRST）→ 锁定执行顺序                         │
  │                                                          │
  │  四重语用加压 = 裁量空间趋近于零                        │
  └──────────────────────────────────────────────────────────┘
```
