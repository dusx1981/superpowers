# 04 横切技能

横切技能不在主链中，但在任何环节都可能被触发。它们由 Controller 动态判断，而不是由唯一出口驱动。

## 三种横切技能

```
  ┌──────────────────────────────────────────────────────────┐
  │  横切技能        触发条件              作用              │
  │  ──────────      ──────────           ────               │
  │  systematic-     遇到 bug/测试失败     切入调试流程      │
  │  debugging       (任何环节都可能)                         │
  │                                                          │
  │  verification-   声称完成/修复/通过     切入验证流程      │
  │  before-         (任何环节都可能)                         │
  │  completion                                                │
  │                                                          │
  │  receiving-      收到审阅反馈           切入反馈处理      │
  │  code-review     (审阅环节)                                │
  └──────────────────────────────────────────────────────────┘
```

## 横切 vs 主链的交互

```
  ┌──────────────────────────────────────────────────────────┐
  │  主链（纵向）：                                          │
  │  brainstorming → writing-plans → subagent → finishing     │
  │  → 每个 AI 消息都沿着主链向前推进                       │
  │  → 顺序锁定保证不遗漏                                  │
  │                                                          │
  │  横切（横向）：                                          │
  │  在主链的任何节点，如果条件满足，切入横切技能            │
  │  → 不是"下一步"，而是"当前步骤的异常处理"             │
  │  → 横切技能处理后，回到主链继续                        │
  │                                                          │
  │  交互示意：                                             │
  │                                                          │
  │  brainstorming → writing-plans → subagent                  │
  │                       │          │                        │
  │                       │          ├→ TDD (测试失败)        │
  │                       │          ├→ debugging (bug)       │
  │                       │          ├→ verification (声称完) │
  │                       │          └→ receiving (审阅反馈)  │
  │                       │                                   │
  │                       └→ verification (计划声称完成)      │
  │                                                          │
  │  → 主链像高速公路，横切像匝道                          │
  │  → 正常情况走主链，异常情况走横切                      │
  │  → 横切处理完回到主链继续                              │
  └──────────────────────────────────────────────────────────┘
```

## Controller 如何判断横切触发

```
  ┌──────────────────────────────────────────────────────────┐
  │  using-superpowers 的流程图在每个用户消息时执行：        │
  │                                                          │
  │  "Might any skill apply?"                                │
  │  → yes, even 1% → Invoke Skill tool                      │
  │  → definitely not → Respond                              │
  │                                                          │
  │  这个判断在主链的每一步都会执行：                       │
  │                                                          │
  │  Step 1: brainstorming 进行中                             │
  │  → 用户消息："先做设计再说"                             │
  │  → "Might any skill apply?"                               │
  │  → 主链技能 (brainstorming) → 继续                      │
  │  → 横切技能 (无触发条件) → 不调用                      │
  │                                                          │
  │  Step 2: subagent-driven 进行中                           │
  │  → 子智能体报告：测试失败                                │
  │  → "Might any skill apply?"                               │
  │  → 横切技能 (debugging) → 触发！                        │
  │  → 切入 debugging 流程                                   │
  │  → debugging 完成 → 回到 subagent-driven                │
  │                                                          │
  │  Step 3: 实现完成                                        │
  │  → AI 说："任务完成了"                                  │
  │  → "Might any skill apply?"                               │
  │  → 横切技能 (verification) → 触发！                     │
  │  → 切入 verification 流程                                │
  │  → verification 通过 → 回到 subagent-driven             │
  │                                                          │
  │  关键：Controller 不需要"预知"哪个横切技能会被触发     │
  │  → 它在每一步都检查"是否有技能适用"                    │
  │  → 横切技能的触发是动态的、上下文相关的                │
  │  → 不是静态的流程定义                                   │
  └──────────────────────────────────────────────────────────┘
```

## 横切技能的入口门控

```
  ┌──────────────────────────────────────────────────────────┐
  │  横切技能不是"任何时候都能调用"——                      │
  │  它们有自己的入口门控（触发条件）。                      │
  │                                                          │
  │  debugging 的入口门控：                                  │
  │  description: "Use when encountering any bug,             │
  │   test failure, or unexpected behavior,                   │
  │   before proposing fixes"                                │
  │  → 只在遇到 bug/失败时触发                              │
  │  → 不是"可以用来探索代码"                              │
  │                                                          │
  │  verification 的入口门控：                               │
  │  description: "Use when about to claim work is complete,  │
  │   fixed, or passing, before committing or creating PRs"  │
  │  → 只在声称完成时触发                                   │
  │  → 不是"可以用来检查代码风格"                          │
  │                                                          │
  │  receiving-code-review 的入口门控：                       │
  │  description: "Use when receiving code review feedback"   │
  │  → 只在收到审阅反馈时触发                               │
  │  → 不是"可以用来请求审阅"                              │
  │                                                          │
  │  入口门控 = 横切技能的"护栏"                           │
  │  → 防止横切技能在不恰当的时机被调用                   │
  │  → 确保横切技能只在它们专长的场景中生效                │
  │  → 角色分工在横切维度也适用                            │
  └──────────────────────────────────────────────────────────┘
```

## 横切技能与主链技能的协同

```
  ┌──────────────────────────────────────────────────────────┐
  │  debugging + TDD 的协同：                                │
  │  ────────────────────                                    │
  │  主链中 TDD 执行 → 测试失败                              │
  │  → 触发 debugging → 找到根因                            │
  │  → debugging 说"写失败测试复现 bug"                     │
  │  → 回到 TDD → 写失败测试 → 实现 → 通过                 │
  │  → debugging + TDD 形成闭环                             │
  │                                                          │
  │  verification + TDD 的协同：                             │
  │  ──────────────────────                                  │
  │  TDD 完成一个功能 → AI 想声称完成                       │
  │  → 触发 verification → 运行测试 → 确认通过              │
  │  → verification 说"证据充分"                            │
  │  → 继续主链                                             │
  │  → verification = TDD 的出口检查                        │
  │                                                          │
  │  receiving-code-review + verification 的协同：            │
  │  ──────────────────────────────────────────               │
  │  审阅者给出反馈 → 触发 receiving-code-review             │
  │  → 处理反馈，修改代码                                   │
  │  → 修改完成 → 触发 verification                         │
  │  → 确认修改确实修复了问题                               │
  │  → receiving-code-review + verification = 审阅闭环       │
  │                                                          │
  │  协同模式：                                             │
  │  主链技能推进流程，横切技能处理异常                     │
  │  → 主链 = 正常路径                                      │
  │  → 横切 = 异常处理路径                                  │
  │  → 两者交替 = 完整的工作流                              │
  └──────────────────────────────────────────────────────────┘
```
