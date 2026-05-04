# 技能协同：技术分析

Superpowers 有 15+ 个技能，每个独立定义了流程和约束。但技能之间如何协同成一个大于部分之和的系统？本系列分析三层协调机制、上下文构造、以及 Controller 的协调角色。

## 核心命题

```
  ┌──────────────────────────────────────────────────────────┐
  │  单个技能各自有效，但它们的价值倍增来自协同。          │
  │                                                          │
  │  协调解决三个问题：                                      │
  │  1. 技能冲突——两个技能要求不同的事                     │
  │  2. 技能遗漏——AI 跳过某个应该调用的技能                │
  │  3. 技能倒序——AI 调用技能的顺序不对                   │
  │                                                          │
  │  Superpowers 的三层协调：                                │
  │  Layer 1: 顺序锁定——每个技能声明唯一出口               │
  │  Layer 2: 角色分工——每个技能只做一件事                 │
  │  Layer 3: 嵌套调用——技能内部调用其他技能               │
  │                                                          │
  │  协调的执行机制：上下文构造                              │
  │  → Controller 为每个技能定制不同的信息环境              │
  │  → 三种模式：技能注入 / 子智能体派发 / 启动注入        │
  │  → 五个模式：信息最小化 / 偏见隔离 / 预设立场          │
  │             / 信息转换 / 生命周期管理                    │
  └──────────────────────────────────────────────────────────┘
```

## 文档索引

### 第一部分：协调机制

| 文件 | 核心内容 |
|------|---------|
| [01-sequence-lock.md](01-sequence-lock.md) | 唯一出口设计：brainstorming→writing-plans→subagent→finishing |
| [02-role-division.md](02-role-division.md) | 每个技能只做一件事：定义/计划/执行/验证/收尾 |
| [03-nested-calls.md](03-nested-calls.md) | 嵌套调用：subagent 内部调用 TDD、审阅、finishing |
| [04-cross-cutting.md](04-cross-cutting.md) | 横切技能：debugging/verification 不在主链但无处不在 |
| [05-controller-role.md](05-controller-role.md) | Controller 是管道连接器，不是中央调度器 |
| [06-pipeline-walkthrough.md](06-pipeline-walkthrough.md) | 完整走读：一个功能从想法到合并的技能协同全链路 |

### 第二部分：上下文构造

| 文件 | 核心内容 |
|------|---------|
| [07-context-construction.md](07-context-construction.md) | 上下文构造总论：七个维度、三种模式 |
| [08-skill-injection.md](08-skill-injection.md) | 模式一：技能注入——主链的上下文构造（七维度详解） |
| [09-subagent-dispatch.md](09-subagent-dispatch.md) | 模式二：子智能体派发——隔离的上下文构造（三种子智能体对比） |
| [10-bootstrap-injection.md](10-bootstrap-injection.md) | 模式三：启动注入——会话诞生的上下文构造（插件机制） |
| [11-key-patterns.md](11-key-patterns.md) | 五个跨模式关键设计模式：最小化/隔离/预设/转换/生命周期 |

### 第三部分：Controller 逻辑

| 文件 | 核心内容 |
|------|---------|
| [12-controller-design-philosophy.md](12-controller-design-philosophy.md) | Controller 的五个设计原则：声明式/约束/协议/隔离/冗余 |
| [13-protocol-design.md](13-protocol-design.md) | 四层协议栈：会话/路由/切换/子智能体通信 + 唯一出口/横切触发/消息格式规范 |
| [14-context-management.md](14-context-management.md) | 上下文四种状态（空白/初始/累积/隔离）+ 五种管理策略 + 跨会话边界 |
| [15-subagent-context-logic.md](15-subagent-context-logic.md) | 三种子智能体的精确上下文构造逻辑：输入/处理/输出/关键决策 |
| [16-fragility-and-improvement.md](16-fragility-and-improvement.md) | 六个脆弱点 + 冗余设计的数学模型 + 代码保证的四个改进方向 |
