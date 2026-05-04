# 02 全大写铁律

全大写（ALL CAPS）是结构性权重中**最直观的格式控制**——在等宽字体和语法高亮中，全大写文本天然醒目，AI 几乎不可能视而不见。

## 5 条铁律

### 2.1 TDD 铁律

**来源**：`skills/test-driven-development/SKILL.md:32-35`

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

结构特征：全大写 + 代码块格式 + 独立成段 + 否定句式 + 时序条件

### 2.2 验证铁律

**来源**：`skills/verification-before-completion/SKILL.md:17-20`

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

结构特征：全大写 + 代码块格式 + 独立成段 + 否定句式 + 证据条件

### 2.3 调试铁律

**来源**：`skills/systematic-debugging/SKILL.md:17-20`

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

结构特征：全大写 + 代码块格式 + 独立成段 + 否定句式 + 过程条件

### 2.4 技能测试铁律

**来源**：`skills/writing-skills/SKILL.md:374-378`

```
NO SKILL WITHOUT A FAILING TEST FIRST
```

结构特征：全大写 + 代码块格式 + 与 TDD 铁律对称

### 2.5 测试反模式铁律

**来源**：`skills/test-driven-development/testing-anti-patterns.md:13-19`

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

结构特征：全大写 + 编号列表 + 否定句式

## 铁律的共同结构模式

```
  ┌──────────────────────────────────────────────────────────┐
  │  铁律的统一格式：                                        │
  │                                                          │
  │  ## The Iron Law                  ← 独立标题，暗示不可逾越 │
  │                                                          │
  │  ```                               ← 代码块包裹           │
  │  NO X WITHOUT Y FIRST             ← 全大写 + 否定 + 条件  │
  │  ```                                                     │
  │                                                          │
  │  铁律公式：NO [禁止的行为] WITHOUT [必须的前提] FIRST      │
  │                                                          │
  │  四个要素：                                              │
  │  1. NO — 否定词，建立禁止                                 │
  │  2. X — 禁止的行为（写代码/声称完成/修复/发布技能）       │
  │  3. WITHOUT — 条件词，建立依赖                            │
  │  4. Y FIRST — 必须的前提（失败测试/验证证据/根因调查）    │
  └──────────────────────────────────────────────────────────┘
```

## ALL CAPS 的强度梯度

```
  同一个意思的不同表达方式：
  ──────────────────────────

  "You should write tests first"           → 建议性（低）
  "Write tests first"                      → 命令性（中）
  "You MUST write tests first"             → 强制性（高）
  "NEVER write code before tests"          → 禁止性（很高）
  "NO CODE WITHOUT TEST FIRST"             → 铁律（最高）

  关键差异：
  - 建议/命令：AI 可以考虑是否遵循
  - MUST：AI 应该遵循但有裁量空间
  - NEVER/NO：AI 不应考虑是否遵循，直接服从
  - 代码块 + 全大写：格式本身拒绝忽视
```

## ALL CAPS 的出现位置统计

| 技能 | ALL CAPS 次数 | 最强表达 |
|------|-------------|---------|
| using-superpowers | 4 | "ABSOLUTELY MUST" |
| test-driven-development | 8 | "NO PRODUCTION CODE WITHOUT..." |
| verification-before-completion | 6 | "NO COMPLETION CLAIMS WITHOUT..." |
| systematic-debugging | 5 | "NO FIXES WITHOUT..." |
| brainstorming | 2 | "You MUST create a task..." |
| subagent-driven-development | 3 | "Never ignore an escalation..." |
| writing-skills | 3 | "NO SKILL WITHOUT..." |
| receiving-code-review | 2 | "NEVER" |
| writing-plans | 1 | "REQUIRED SUB-SKILL" |

## "MANDATORY" 标注

除铁律外，一些关键步骤用 MANDATORY 强调：

```
  TDD RED 阶段验证：
  "Verify RED - Watch It Fail
   MANDATORY. Never skip."

  TDD GREEN 阶段验证：
  "Verify GREEN - Watch It Pass
   MANDATORY."

  调试阶段完成：
  "You MUST complete each phase
   before proceeding to the next."

  头脑风暴清单：
  "You MUST create a task for each
   of these items and complete them in order"
```

## ALL CAPS 为什么有效？

```
  ┌──────────────────────────────────────────────────────┐
  │  LLM 处理 ALL CAPS 的三个机制：                     │
  │                                                      │
  │  1. 训练数据关联                                     │
  │     ALL CAPS 在训练语料中通常出现在：                │
  │     - 法律条文（"SHALL NOT"）                       │
  │     - 安全警告（"DO NOT TOUCH"）                    │
  │     - 军事指令（"DO NOT FIRE"）                     │
  │     - 紧急公告（"STOP"）                            │
  │     → AI 学会了 ALL CAPS = 不可忽视                 │
  │                                                      │
  │  2. 注意力加权                                       │
  │     ALL CAPS 文本在 token 序列中更突出：             │
  │     - 与周围小写文本形成对比                         │
  │     - 每个字符都是"大写 token"                      │
  │     - 在注意力机制中获得更高权重                      │
  │                                                      │
  │  3. 语义强化                                         │
  │     "MUST" 比 "must" 多了情感强度                    │
  │     "NEVER" 比 "never" 多了绝对性                    │
  │     "NO X WITHOUT Y" 比 "don't do X without Y"      │
  │     更像法律条文而非建议                             │
  └──────────────────────────────────────────────────────┘
```
