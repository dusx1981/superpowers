# Superpowers 工作流控制：技术分析

Superpowers 控制工作流的本质不是代码层面的流程引擎，而是**用精心设计的 Prompt 语言对 AI 进行行为塑形**。每一层控制都是在和一个会合理化偷懒的 AI 对话。

## 控制机制全景

```
┌──────────────────────────────────────────────────────────────────────┐
│                   Superpowers 工作流控制全景                          │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  控制层               机制                        例子                │
│  ───────              ────                        ────                │
│                                                                      │
│  1. 注入控制          <EXTREMELY_IMPORTANT>       SessionStart Hook  │
│                       "不可协商"语言              抢先注入技能意识    │
│                                                                      │
│  2. 触发控制          description 字段            "You MUST use      │
│                       1% 规则                     this before..."    │
│                                                                      │
│  3. 门禁控制          <HARD-GATE> 标签            brainstorming 中   │
│                       "绝不" 红旗规则             不得写代码         │
│                                                                      │
│  4. 出口控制          "唯一出口" 声明             brainstorming →    │
│                       禁止自由选择下一步          只能 writing-plans  │
│                                                                      │
│  5. 顺序控制          流程图 + 强制顺序           规范审阅 → 代码审阅│
│                       顺序不可颠倒                不能反过来         │
│                                                                      │
│  6. 质量控制          反模式列表                  "TODO" = 计划失败  │
│                       自审清单                    写完必须自审       │
│                                                                      │
│  7. 信任控制          "不要信任实现者"            spec-reviewer      │
│                       独立验证原则                必须读实际代码     │
│                                                                      │
│  8. 失败控制          四种状态 + 升级链           BLOCKED → 必须改变 │
│                       禁止无条件重试              条件后重新派遣     │
│                                                                      │
│  9. 验证控制          铁律 + "撒谎"强语言        没验证 = 撒谎     │
│                       必须运行命令                不能凭感觉声称     │
│                                                                      │
│  10. 合理化防御       红旗表格                    12 个合理化借口    │
│                       "这些想法 = 停下来"        及其反驳           │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

## 文档索引

| 文件 | 内容 |
|------|------|
| [01-injection-control.md](01-injection-control.md) | 注入控制：SessionStart Hook、EXTREMELY_IMPORTANT 包裹、平台适配 |
| [02-trigger-control.md](02-trigger-control.md) | 触发控制：description 字段匹配、1% 规则、技能调度流程 |
| [03-gate-control.md](03-gate-control.md) | 门禁控制：HARD-GATE、唯一出口、红旗规则 |
| [04-sequence-control.md](04-sequence-control.md) | 顺序控制：流程图强制顺序、审阅链、顺序不可颠倒 |
| [05-quality-control.md](05-quality-control.md) | 质量控制：反模式列表、自审清单、Checklist → TodoWrite |
| [06-trust-control.md](06-trust-control.md) | 信任控制：不信任原则、独立验证、审阅者与实现者的对立设计 |
| [07-failure-control.md](07-failure-control.md) | 失败控制：四种状态、升级链、禁止无条件重试 |
| [08-verification-control.md](08-verification-control.md) | 验证控制：铁律、"撒谎"强语言、必须运行命令 |
| [09-rationalization-defense.md](09-rationalization-defense.md) | 合理化防御：红旗表格、反合理化模式、语言强度梯度 |
| [10-case-study.md](10-case-study.md) | 综合案例：暗黑模式全链路追踪，展示每步控制如何生效 |
