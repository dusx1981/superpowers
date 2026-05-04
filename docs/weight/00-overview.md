# 结构性权重：技术分析

结构性权重是 Superpowers 中让 AI **更难忽略、更难绕过**指令的一切机制。普通文本是"建议"，结构性权重是"铁律"。本系列文档逐一分析所有权重类型、实现方式和作用原理。

## 核心命题

```
  ┌──────────────────────────────────────────────────────────────┐
  │                                                              │
  │  同一个意思，不同的表达 → AI 的服从概率截然不同             │
  │                                                              │
  │  "建议你先测试"           → 服从概率 ~40%                  │
  │  "你应该先测试"           → 服从概率 ~60%                  │
  │  "你必须先测试"           → 服从概率 ~80%                  │
  │  "NO CODE WITHOUT TEST"   → 服从概率 ~95%                  │
  │  <HARD-GATE>不可写代码</HARD-GATE>  → 服从概率 ~98%        │
  │                                                              │
  │  结构性权重 = 让指令从"建议"变成"不可绕过"的一切手段       │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

## 权重类型全景

```
┌──────────────────────────────────────────────────────────────────────┐
│                     结构性权重全景图                                   │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  类型                来源                          详见                │
│  ────                ────                          ────                │
│                                                                      │
│  1. XML 标签包裹     标签本身的结构性              01-xml-tags.md      │
│  2. 全大写铁律       格式 + 语言双重加压           02-iron-laws.md     │
│  3. 否定式语言       "绝不"/"Never" vs "不要"      03-negation.md      │
│  4. 红旗表格         预判合理化 + 逐一批驳         04-red-flags.md     │
│  5. 流程图/状态机    图形化规定顺序和分支           05-flowcharts.md    │
│  6. 清单结构         Checklist → TodoWrite 强追踪  06-checklists.md    │
│  7. 反模式列表       负面清单——写 = 失败           07-anti-patterns.md │
│  8. 结构化状态       四种状态码，非自由文本         08-structured-status.md │
│  9. Prompt 模板      子智能体的角色约束             09-prompt-templates.md │
│  10. 注入时机        SessionStart 抢占先机          10-injection-timing.md │
│  11. 子智能体隔离    上下文不继承，无法绕过         11-subagent-isolation.md │
│  12. 审阅循环        不通过必须重审，不能跳过       12-review-loops.md  │
│  13. 工具映射        跨平台指令转译保证可用性       13-tool-mapping.md  │
│  14. 模型选择        能力匹配防止低能高用           14-model-selection.md │
│  15. 对抗性设计      审阅者与实现者利益对立         15-adversarial-design.md │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

## 权重强度梯度

```
  强度递增（从弱到强）：
  ────────────────────

  Level 0: 普通文本            "建议你..."           AI 可能忽略
  Level 1: 加粗/强调           **你应该...**         AI 注意到但可绕过
  Level 2: 命令式语言          "You MUST..."         AI 不应违背
  Level 3: 否定式语言          "Never..." / "Do NOT" AI 很难绕过
  Level 4: 全大写              "NO X WITHOUT Y"      格式本身即命令
  Level 5: XML 标签            <HARD-GATE>           结构性不可忽略
  Level 6: XML 标签 + 铁律     <EXTREMELY_IMPORTANT>  最高权重
           + ALL CAPS          + "NO CODE WITHOUT TEST"
           + 注入时机优先       + SessionStart 抢先
```

## 文档索引

| 文件 | 核心内容 |
|------|---------|
| [01-xml-tags.md](01-xml-tags.md) | 9 种 XML 标签：EXTREMELY_IMPORTANT、HARD-GATE、SUBAGENT-STOP、Good/Bad 等 |
| [02-iron-laws.md](02-iron-laws.md) | 5 条铁律 + ALL CAPS 强调：TDD、验证、调试、技能测试 |
| [03-negation.md](03-negation.md) | 否定式语言的强度梯度："绝不"/"Never" vs "不要"/"Don't" |
| [04-red-flags.md](04-red-flags.md) | 4 套红旗表格：预判合理化 + 逐一批驳 |
| [05-flowcharts.md](05-flowcharts.md) | Graphviz 流程图：强制顺序、分支条件、回环路径 |
| [06-checklists.md](06-checklists.md) | Checklist → TodoWrite：外部可追踪的步骤控制 |
| [07-anti-patterns.md](07-anti-patterns.md) | 反模式列表：写 = 失败的负面清单 |
| [08-structured-status.md](08-structured-status.md) | 四种状态码 + 结构化报告格式 |
| [09-prompt-templates.md](09-prompt-templates.md) | 子智能体 Prompt 模板：角色约束 + 不信任嵌入 |
| [10-injection-timing.md](10-injection-timing.md) | SessionStart 抢占 + messages.transform 注入时机 |
| [11-subagent-isolation.md](11-subagent-isolation.md) | 上下文不继承：全新实例无法绕过 Controller 控制 |
| [12-review-loops.md](12-review-loops.md) | 审阅循环：不通过 = 修复 + 重新审阅，不能跳过 |
| [13-tool-mapping.md](13-tool-mapping.md) | 跨平台工具映射：技能指令可用性保证 |
| [14-model-selection.md](14-model-selection.md) | 模型选择策略：能力匹配防止低能高用 |
| [15-adversarial-design.md](15-adversarial-design.md) | 对抗性设计：审阅者与实现者利益对立 |
