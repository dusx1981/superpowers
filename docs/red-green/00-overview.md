# 回归测试 Red-Green 验证：技术分析

回归测试的 Red-Green 验证是 Superpowers 验证体系中最精妙的设计。它不是一个流程，而是一个**对照实验**——用科学方法证明"测试确实能检测 bug"。

## 核心命题

```
  ┌──────────────────────────────────────────────────────────────┐
  │  你修了一个 bug，写了一个回归测试。测试通过了。             │
  │                                                              │
  │  你能信任这个测试吗？                                        │
  │                                                              │
  │  不能。                                                      │
  │                                                              │
  │  因为"测试通过"可能意味着：                                 │
  │  1. 测试正确验证了 bug 修复 ← 你期望的                     │
  │  2. 测试碰巧通过（验证了错误的行为）                       │
  │  3. 测试是恒真断言（永远通过）                              │
  │  4. 测试验证了实现细节（重构会破坏它）                      │
  │                                                              │
  │  你怎么知道是哪种？                                          │
  │  → 你必须"看到测试失败"才能确认测试有效。                 │
  │  → 但 bug 已经修了，测试当然通过。                         │
  │  → 除非你故意把 bug 放回去。                                │
  │                                                              │
  │  这就是 Red-Green 验证的核心逻辑。                          │
  └──────────────────────────────────────────────────────────────┘
```

## 五步序列

Superpowers 的五步验证：

```
  Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
```

## 文档索引

| 文件 | 核心内容 |
|------|---------|
| [01-five-steps.md](01-five-steps.md) | 五步序列的每一步证明了什么 |
| [02-controlled-experiment.md](02-controlled-experiment.md) | 对照实验类比、因果推断、科学方法 |
| [03-symmetry-with-tdd.md](03-symmetry-with-tdd.md) | 与 TDD Red-Green 的对称性、核心原则统一 |
| [04-failure-as-proof.md](04-failure-as-proof.md) | "失败"即证明：信息论视角、MUST FAIL 的强制性 |
| [05-without-verification.md](05-without-verification.md) | 不做 Red-Green 验证的风险、四种虚假信心 |
