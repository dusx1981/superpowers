# 01 顺序锁定

顺序锁定是协调的第一层——每个技能声明自己的"唯一出口"，指向下一个技能。AI 没有选择跳到其他技能的裁量空间。

## 唯一出口设计

```
  ┌──────────────────────────────────────────────────────────┐
  │  技能          唯一出口/分支出口                          │
  │  ────          ────────────────                          │
  │  brainstorming  → writing-plans                           │
  │                  "Do NOT invoke any other skill.          │
  │                   writing-plans is the next step."        │
  │                                                          │
  │  writing-plans  → subagent-driven-development (推荐)      │
  │                  → executing-plans (备选)                 │
  │                  "REQUIRED SUB-SKILL"                     │
  │                                                          │
  │  subagent       → finishing-a-development-branch          │
  │                  (通过流程图终态节点指定)                 │
  │                                                          │
  │  finishing      → merge/PR/cleanup (终态)                │
  └──────────────────────────────────────────────────────────┘
```

## "唯一出口"防止的三种错误

### 防止技能遗漏

```
  ┌──────────────────────────────────────────────────────────┐
  │  没有"唯一出口"时：                                    │
  │  brainstorming 完成 → AI 自由选择下一步                  │
  │  → 可能跳过 writing-plans 直接实现                     │
  │  → 可能调用 mcp-builder 而非 writing-plans              │
  │  → 可能什么都不做就停了                                │
  │  → 4 种可能，3 种是错误的                              │
  │                                                          │
  │  有"唯一出口"时：                                      │
  │  brainstorming 完成 → 唯一出口: writing-plans           │
  │  "Do NOT invoke any other skill.                          │
  │   writing-plans is the next step."                       │
  │  → AI 只有 1 种选择                                    │
  │  → 遗漏概率从 75% 降到接近 0%                         │
  │                                                          │
  │  类比：                                                  │
  │  没有唯一出口 = 十字路口 → AI 可能走错方向              │
  │  唯一出口 = 单行道 → AI 只能向前走                     │
  └──────────────────────────────────────────────────────────┘
```

### 防止技能倒序

```
  ┌──────────────────────────────────────────────────────────┐
  │  场景：AI 先实现了代码，然后才想到调用 brainstorming    │
  │                                                          │
  │  brainstorming 的 HARD-GATE 防止了倒序：                 │
  │  "Do NOT invoke any implementation skill,                 │
  │   write any code, scaffold any project,                   │
  │   or take any implementation action                      │
  │   until you have presented a design                      │
  │   and the user has approved it."                         │
  │                                                          │
  │  → 即使 AI 想先写代码，HARD-GATE 拦住了               │
  │  → 必须先设计 → 用户批准 → 才能进入下一技能           │
  │  → 顺序锁定的双向保障：                                 │
  │    - "唯一出口"保证向前不遗漏                          │
  │    - "HARD-GATE"保证不跳过当前步骤                    │
  └──────────────────────────────────────────────────────────┘
```

### 防止技能冲突

```
  ┌──────────────────────────────────────────────────────────┐
  │  场景：AI 同时看到 brainstorming 和 subagent-driven     │
  │  → 不知道该先做设计还是先派子智能体                    │
  │                                                          │
  │  顺序锁定消除了冲突：                                   │
  │  brainstorming 的唯一出口 = writing-plans                │
  │  writing-plans 的出口 = subagent-driven                  │
  │  → 顺序是强制的：brainstorming → writing-plans           │
  │    → subagent-driven                                    │
  │  → AI 不需要"决定"顺序——顺序已经被锁定了             │
  │  → 冲突不存在——因为只有一条路径                       │
  └──────────────────────────────────────────────────────────┘
```

## 主链的完整技能序列

```
  ┌──────────────────────────────────────────────────────────┐
  │  Superpowers 的主链：                                    │
  │                                                          │
  │  using-superpowers (入口)                                 │
  │       │                                                  │
  │       ▼                                                  │
  │  brainstorming (设计)                                    │
  │       │ 唯一出口                                         │
  │       ▼                                                  │
  │  writing-plans (计划)                                    │
  │       │ 分支出口                                         │
  │       ├────────→ subagent-driven-development (推荐)       │
  │       │             │ 唯一出口                            │
  │       │             ▼                                    │
  │       │         finishing-a-development-branch (收尾)      │
  │       │                                                  │
  │       └────────→ executing-plans (备选)                   │
  │                     │                                    │
  │                     ▼                                    │
  │                 finishing-a-development-branch (收尾)      │
  │                                                          │
  │  两条路径共享终态：finishing-a-development-branch        │
  │  → 无论走哪条路径，最终都到达同一个收尾技能            │
  │  → 终态一致性                                            │
  └──────────────────────────────────────────────────────────┘
```

## "唯一出口"与裁量空间的关系

```
  ┌──────────────────────────────────────────────────────────┐
  │  "唯一出口"是对裁量空间的另一种压缩。                  │
  │                                                          │
  │  没有唯一出口：                                          │
  │  → AI 在"选哪个技能"上有 100% 裁量空间                │
  │  → 可能选错、遗漏、倒序                                │
  │                                                          │
  │  有唯一出口：                                            │
  │  → AI 在"选哪个技能"上有 0% 裁量空间                  │
  │  → 只有唯一选择，不能选错                              │
  │                                                          │
  │  有分支出口（如 writing-plans 的两个选项）：             │
  │  → AI 在"选哪个技能"上有有限裁量空间                  │
  │  → 只能在两个合法选项中选择                            │
  │  → 裁量空间从"无限选择"压缩到"有限选择"              │
  │                                                          │
  │  顺序锁定 = 在技能选择层面消灭裁量空间                  │
  │  → AI 不需要"判断"该调用哪个技能                      │
  │  → 技能自己告诉 AI 下一步是什么                        │
  │  → 判断的裁量空间 = 0                                   │
  └──────────────────────────────────────────────────────────┘
```
