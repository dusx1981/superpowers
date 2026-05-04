# TDD 原理：技术分析

TDD 不是一个编码流程，它是一个认知模式的切换。本系列从原理层面分析 TDD 为什么有效——不是"怎么用"，而是"为什么"。

## 核心命题

```
  ┌──────────────────────────────────────────────────────────────┐
  │  先写测试 vs 后写测试，表面上只是顺序不同，                │
  │  但这个顺序差异改变了三件事：                                │
  │                                                              │
  │  1. 目标函数——从"怎么实现"变成"实现什么"                  │
  │  2. 信息状态——从"不确定测试是否有效"变成"确认测试有效"    │
  │  3. 反馈回路——从"宏循环（小时级）"变成"微循环（秒级）"    │
  │                                                              │
  │  三个维度同时改变 = TDD 的效果不是线性改进，是范式转换      │
  └──────────────────────────────────────────────────────────────┘
```

## 文档索引

| 文件 | 核心内容 |
|------|---------|
| [01-cognitive-shift.md](01-cognitive-shift.md) | 认知模式切换：RED/GREEN/REFACTOR 对应三种思维 |
| [02-information-theory.md](02-information-theory.md) | 信息论视角：测试的信息流向、实现污染、失败证明 |
| [03-feedback-loops.md](03-feedback-loops.md) | 反馈回路：微循环 vs 宏循环、bug 发现时间、修复成本 |
| [04-delete-means-delete.md](04-delete-means-delete.md) | 删除原则：实现偏见、路径依赖、沉没成本 |
| [05-minimal-code.md](05-minimal-code.md) | 最小代码：YAGNI 的数学基础、预判性编码的概率陷阱 |
| [06-anti-patterns-deep.md](06-anti-patterns-deep.md) | 反模式深析：测试 mock、测试耦合、恒真断言 |
| [07-why-order-matters.md](07-why-order-matters.md) | 顺序为什么重要：Superpowers 五大论点的原理拆解 |

## 与 Superpowers 的关系

```
  docs/TDD/ 系列：分析 TDD 为什么有效（原理）
  skills/test-driven-development/：定义 TDD 怎么做（规范）

  TDD 系列是"原理解析"
  SKILL.md 是"操作手册"

  两者的关系：
  - SKILL.md 的 Iron Law → 07-why-order-matters.md 解释为什么
  - SKILL.md 的 "Delete means delete" → 04-delete-means-delete.md 解释为什么
  - SKILL.md 的 "Minimal code" → 05-minimal-code.md 解释为什么
  - SKILL.md 的 "Watch it fail" → 02-information-theory.md 解释为什么
  - SKILL.md 的 Red-Green-Refactor → 01-cognitive-shift.md 解释为什么
  - testing-anti-patterns.md → 06-anti-patterns-deep.md 解释为什么
```
