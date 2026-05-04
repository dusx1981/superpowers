# 06 完整走读：一个功能从想法到合并

走读一个完整功能开发流程，展示三层协调机制如何在实际工作中协同运作。

## 场景

```
  ┌──────────────────────────────────────────────────────────┐
  │  用户需求："给项目添加一个配置文件验证功能"             │
  │                                                          │
  │  起点：空对话                                            │
  │  终点：代码合并到主分支                                  │
  │  经过：7 个技能，6 次技能切换，3 次横切触发             │
  └──────────────────────────────────────────────────────────┘
```

## Step 1: using-superpowers → brainstorming

```
  ┌──────────────────────────────────────────────────────────┐
  │  用户消息："给项目添加一个配置文件验证功能"             │
  │                                                          │
  │  Controller 判断：                                       │
  │  "Might any skill apply?" → yes                          │
  │  → 加载 using-superpowers                                │
  │  → using-superpowers 流程图：                            │
  │    "About to EnterPlanMode?" → yes                       │
  │    "Already brainstormed?" → no                           │
  │    → "Invoke brainstorming skill"                        │
  │                                                          │
  │  协调机制：                                              │
  │  - 顺序锁定：using-superpowers 的流程图指定              │
  │    brainstorming 为第一个技能                            │
  │  - 角色分工：using-superpowers 只做匹配，不做设计       │
  │                                                          │
  │  brainstorming 开始：                                    │
  │  → HARD-GATE 生效：                                     │
  │    "Do NOT write any code, scaffold any project,          │
  │     or take any implementation action"                   │
  │  → 探索用户意图                                         │
  │  → 产出：设计文档（做什么）                             │
  └──────────────────────────────────────────────────────────┘
```

## Step 2: brainstorming → writing-plans

```
  ┌──────────────────────────────────────────────────────────┐
  │  brainstorming 完成，设计文档已产出                      │
  │                                                          │
  │  唯一出口触发：                                          │
  │  "Do NOT invoke any other skill.                          │
  │   writing-plans is the next step."                       │
  │  → Controller 加载 writing-plans                         │
  │                                                          │
  │  协调机制：                                              │
  │  - 顺序锁定：唯一出口 = writing-plans                    │
  │  - 角色分工：brainstorming 只做设计，不做计划            │
  │    writing-plans 只做计划，不做设计                      │
  │  - 信息封装：brainstorming 产出设计文档，                │
  │    writing-plans 消费设计文档作为输入                    │
  │                                                          │
  │  writing-plans 开始：                                    │
  │  → 消费设计文档                                         │
  │  → 运行自审清单                                         │
  │  → 产出：实现计划（怎么做）                             │
  │  → 计划头声明：                                         │
  │    "REQUIRED SUB-SKILL:                                  │
  │     subagent-driven-development (recommended)"           │
  └──────────────────────────────────────────────────────────┘
```

## Step 3: writing-plans → subagent-driven-development

```
  ┌──────────────────────────────────────────────────────────┐
  │  writing-plans 完成，实现计划已产出                      │
  │                                                          │
  │  分支出口触发：                                          │
  │  计划头声明 subagent-driven-development（推荐）          │
  │  → Controller 加载 subagent-driven-development           │
  │                                                          │
  │  协调机制：                                              │
  │  - 顺序锁定：分支出口 → subagent-driven（推荐路径）      │
  │  - 角色分工：writing-plans 只做计划，不做执行            │
  │  - 嵌套调用：subagent-driven 内部将调用 TDD、审阅等      │
  │                                                          │
  │  subagent-driven-development 开始：                      │
  │  → 解析计划中的任务列表                                 │
  │  → 为每个任务创建实现者子智能体                         │
  └──────────────────────────────────────────────────────────┘
```

## Step 4: subagent 内部 — TDD + 横切触发

```
  ┌──────────────────────────────────────────────────────────┐
  │  实现者子智能体执行任务 1："添加配置解析器"              │
  │                                                          │
  │  嵌套调用：                                              │
  │  实现者内部使用 TDD → Red-Green-Refactor                 │
  │                                                          │
  │  ┌─ TDD Red ──────────────────────────────────────┐     │
  │  │  写测试：test_config_parser_valid_input()       │     │
  │  │  运行测试：FAIL (函数不存在)                    │     │
  │  └────────────────────────────────────────────────┘     │
  │                                                          │
  │  ┌─ TDD Green ────────────────────────────────────┐     │
  │  │  写最小实现：function parseConfig(input) {      │     │
  │  │    return JSON.parse(input)                     │     │
  │  │  }                                              │     │
  │  │  运行测试：PASS                                 │     │
  │  └────────────────────────────────────────────────┘     │
  │                                                          │
  │  ┌─ TDD Refactor ─────────────────────────────────┐     │
  │  │  重构：添加错误处理、类型检查                   │     │
  │  │  运行测试：PASS                                 │     │
  │  └────────────────────────────────────────────────┘     │
  │                                                          │
  │  横切触发 1: AI 声称"任务 1 完成"                       │
  │  → verification-before-completion 触发                   │
  │  → 运行完整测试套件 → 全部通过                         │
  │  → 验证通过 → 继续下一个任务                           │
  │                                                          │
  │  横切触发 2: 任务 2 中测试失败                           │
  │  → systematic-debugging 触发                             │
  │  → 找到根因：配置文件路径编码问题                       │
  │  → 写失败测试复现 → 修复 → 通过                         │
  │  → debugging 完成 → 回到 TDD 继续任务 2                 │
  └──────────────────────────────────────────────────────────┘
```

## Step 5: subagent 内部 — 审阅循环

```
  ┌──────────────────────────────────────────────────────────┐
  │  所有任务实现完成                                       │
  │                                                          │
  │  嵌套调用：审阅循环                                      │
  │                                                          │
  │  ┌─ 规范审阅 ─────────────────────────────────────┐     │
  │  │  Controller 创建规范审阅者子智能体               │     │
  │  │  传入：规范文本 + 实现代码 (BASE_SHA..HEAD_SHA) │     │
  │  │  审阅者判断：代码是否匹配规范？                 │     │
  │  │  → 发现：缺少对空配置文件的处理                 │     │
  │  │  → 反馈：需要添加空配置处理                     │     │
  │  └────────────────────────────────────────────────┘     │
  │                                                          │
  │  横切触发 3: 收到审阅反馈                                │
  │  → receiving-code-review 触发                            │
  │  → 不盲目接受反馈，而是验证：                           │
  │    "空配置是否在规范范围内？"                           │
  │  → 确认：规范确实要求处理空配置                         │
  │  → 实现者添加空配置处理                                 │
  │  → verification 触发 → 确认修复                         │
  │                                                          │
  │  ┌─ 代码审阅 ─────────────────────────────────────┐     │
  │  │  Controller 创建代码审阅者子智能体               │     │
  │  │  传入：实现代码 + 审阅范围                       │     │
  │  │  审阅者判断：代码质量如何？                     │     │
  │  │  → 发现：错误处理可以更健壮                     │     │
  │  │  → 反馈：建议使用 Result 类型替代 throw          │     │
  │  └────────────────────────────────────────────────┘     │
  │                                                          │
  │  receiving-code-review 再次触发                          │
  │  → 验证反馈：Result 类型是否适合当前项目？              │
  │  → 确认：项目已有 Result 模式使用                       │
  │  → 实现者采纳建议，重构错误处理                         │
  │  → verification 触发 → 确认重构不破坏测试               │
  └──────────────────────────────────────────────────────────┘
```

## Step 6: subagent-driven → finishing-a-development-branch

```
  ┌──────────────────────────────────────────────────────────┐
  │  所有审阅通过，所有验证通过                              │
  │                                                          │
  │  终态节点触发：                                          │
  │  subagent-driven-development 流程图的终态 =               │
  │    finishing-a-development-branch                         │
  │  → Controller 加载 finishing-a-development-branch         │
  │                                                          │
  │  协调机制：                                              │
  │  - 顺序锁定：终态节点 = finishing                        │
  │  - 角色分工：subagent 只执行，finishing 只收尾            │
  │  - 信息封装：finishing 只看到分支状态，                  │
  │    不需要知道实现细节                                    │
  │                                                          │
  │  finishing-a-development-branch 开始：                    │
  │  → 检查工作树状态                                       │
  │  → 提供三个选项：                                       │
  │    1. Merge to main                                      │
  │    2. Create Pull Request                                │
  │    3. Cleanup branch                                     │
  │  → 用户选择                                             │
  │  → 执行并完成                                           │
  └──────────────────────────────────────────────────────────┘
```

## 协同机制汇总

```
  ┌──────────────────────────────────────────────────────────┐
  │  三层协调 + 横切的完整交互：                            │
  │                                                          │
  │  顺序锁定（7 次技能切换）：                              │
  │  using-superpowers → brainstorming                       │
  │  brainstorming → writing-plans                           │
  │  writing-plans → subagent-driven                         │
  │  subagent-driven → finishing                             │
  │  → 4 次主链切换，全部由唯一出口/分支出口驱动            │
  │                                                          │
  │  角色分工（0 次冲突）：                                  │
  │  每个技能只做自己职责范围内的事                          │
  │  → 无跨界、无冲突、无遗漏                               │
  │                                                          │
  │  嵌套调用（3 层深度）：                                  │
  │  subagent-driven → TDD → verification                    │
  │  subagent-driven → 规范审阅 → receiving-code-review      │
  │  subagent-driven → 代码审阅 → receiving-code-review      │
  │  → 3 层嵌套，每层独立运行                               │
  │                                                          │
  │  横切触发（3 次）：                                      │
  │  1. verification: AI 声称完成时触发                      │
  │  2. debugging: 测试失败时触发                            │
  │  3. receiving-code-review: 收到审阅反馈时触发            │
  │  → 3 次横切，每次都是动态判断触发                       │
  │                                                          │
  │  总计：                                                  │
  │  - 7 个技能参与                                          │
  │  - 4 次主链切换（顺序锁定）                              │
  │  - 3 次横切触发（动态协调）                              │
  │  - 3 层嵌套（信息封装）                                  │
  │  - 0 次冲突（角色分工）                                  │
  │  - 0 次遗漏（唯一出口）                                  │
  │  - 0 次倒序（HARD-GATE）                                 │
  └──────────────────────────────────────────────────────────┘
```
