# 注意力稀释与注意力焦点：技术分析

同一个 AI 模型，在不同的上下文中，表现出截然不同的行为。这不是模型的缺陷——这是模型的特性。Superpowers 的整个技能系统，本质上是一套注意力工程：在 AI 的有限注意力带宽中，将焦点精确导向当前任务所需的行为规则。

## 核心命题

```
  ┌──────────────────────────────────────────────────────────┐
  │  AI 的注意力是稀缺资源。                                │
  │                                                          │
  │  上下文越长，AI 对任意单条指令的服从概率越低。          │
  │  指令越分散，AI 的行为越不可预测。                      │
  │  Superpowers 的设计 = 注意力稀缺条件下的工程学。        │
  │                                                          │
  │  注意力工程的三个子问题：                                │
  │  1. 注意力捕获——如何在长上下文中让关键指令被"看见"      │
  │  2. 注意力保持——如何防止关键指令在后续交互中被"遗忘"    │
  │  3. 注意力切换——如何在新技能加载时重新锚定焦点          │
  └──────────────────────────────────────────────────────────┘
```

## 文档索引

| 文件 | 核心内容 |
|------|---------|
| [01-attention-scarcity.md](01-attention-scarcity.md) | 注意力稀缺性：为什么 AI 的注意力是有限资源 |
| [02-capture-mechanisms.md](02-capture-mechanisms.md) | 注意力捕获：7 种机制确保关键指令被"看见" |
| [03-retention-mechanisms.md](03-retention-mechanisms.md) | 注意力保持：冗余、重复、首尾锚定 |
| [04-switching-mechanisms.md](04-switching-mechanisms.md) | 注意力切换：Announce、唯一出口、流程图 |
| [05-dilution-model.md](05-dilution-model.md) | 注意力稀释模型：量化与预测 |
| [06-skill-ordering.md](06-skill-ordering.md) | 技能内容排序：先约束后自由的认知设计 |
| [07-iron-law-pattern.md](07-iron-law-pattern.md) | 铁律模式：最高注意力权重的结构设计 |
| [08-rationalization-defense.md](08-rationalization-defense.md) | 反理性化防御：红旗表和借口反驳表 |
