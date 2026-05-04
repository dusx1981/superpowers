# 02 触发控制

触发控制决定 AI **何时**使用哪个技能。核心机制是：description 字段自动匹配 + 1% 规则强制检查。

## 双层触发机制

```
  ┌──────────────────────────────────────────────────────────────┐
  │                      双层触发                                 │
  │                                                              │
  │  Layer 1: 自动触发                                           │
  │  ┌────────────────────────────────────────────────────────┐  │
  │  │  平台扫描技能的 description 字段                       │  │
  │  │  当用户消息匹配 description 中的触发条件时             │  │
  │  │  自动建议/激活技能                                     │  │
  │  └────────────────────────────────────────────────────────┘  │
  │                                                              │
  │  Layer 2: 强制触发                                           │
  │  ┌────────────────────────────────────────────────────────┐  │
  │  │  using-superpowers 的 1% 规则：                        │  │
  │  │  "哪怕只有 1% 的可能性技能适用，你也必须调用"          │  │
  │  │  → 覆盖了 AI 可能的"这不需要技能"判断                │  │
  │  └────────────────────────────────────────────────────────┘  │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

## description 字段：技能的"触发条件"

每个技能的 YAML frontmatter 中都有 `description` 字段，它定义了**何时应该使用这个技能**：

```yaml
# brainstorming
description: "You MUST use this before any creative work - creating features,
  building components, adding functionality, or modifying behavior."

# writing-plans
description: "Use when you have a spec or requirements for a multi-step task,
  before touching code"

# subagent-driven-development
description: "Use when executing implementation plans with independent tasks
  in the current session"

# systematic-debugging
description: "Use when encountering any bug, test failure, or unexpected
  behavior, before proposing fixes"

# test-driven-development
description: "Use when implementing any feature or bugfix, before writing
  implementation code"

# verification-before-completion
description: "Use when about to claim work is complete, fixed, or passing,
  before committing or creating PRs"
```

### description 的语言强度梯度

```
  强度递增：
  ────────

  "Use when..."          ← 建议性（低）
  "Use when X, before Y" ← 时序性（中）
  "You MUST use this..." ← 命令性（高）
  "...before ANY response" ← 绝对性（最高）
```

| 技能 | description 语言强度 | 原因 |
|------|---------------------|------|
| `using-superpowers` | 最高（"1% chance"） | 这是所有技能的入口，必须最强制 |
| `brainstorming` | 高（"You MUST"） | 防止跳过设计直接实现 |
| `test-driven-development` | 高（"before writing implementation"） | 防止先写代码后补测试 |
| `systematic-debugging` | 高（"before proposing fixes"） | 防止猜着修 bug |
| `verification-before-completion` | 高（"before committing"） | 防止未验证就声称完成 |
| `writing-plans` | 中（"Use when you have a spec"） | 条件触发，不强制 |
| `dispatching-parallel-agents` | 低（"Use when facing 2+ independent tasks"） | 场景限定 |

## 1% 规则：兜底触发

`using-superpowers` 技能中的 1% 规则是**兜底机制**——即使 description 自动匹配没触发，AI 也必须主动检查：

```
  ┌──────────────────────────────────────────────────────┐
  │  using-superpowers 技能内容：                         │
  │                                                      │
  │  "Invoke relevant or requested skills BEFORE any     │
  │   response or action. Even a 1% chance a skill      │
  │   might apply means that you should invoke the       │
  │   skill to check."                                   │
  │                                                      │
  │  翻译：                                              │
  │  在任何响应或行动之前调用相关技能。                   │
  │  哪怕只有 1% 的可能性技能适用，                       │
  │  你也应该调用技能来检查。                             │
  └──────────────────────────────────────────────────────┘
```

### 为什么需要 1% 规则？

没有 1% 规则时，AI 会合理化跳过技能：

```
  AI 的内心独白（无 1% 规则）：
  ──────────────────────────
  "用户只是让我加个暗黑模式，这很简单，
   不需要 brainstorming 那么重的流程..."
  → 跳过技能，直接写代码
  → 结果：缺少设计，返工

  AI 的内心独白（有 1% 规则）：
  ────────────────────────
  "用户让我加暗黑模式...这是创造性工作，
   brainstorming 的 description 说 'You MUST use
   this before any creative work'...
   哪怕只有 1% 可能适用，我也必须调用..."
  → 调用 brainstorming
  → 结果：先设计后实现
```

## 技能调度流程

```
  用户消息到达
       │
       ▼
  ┌─────────────────┐
  │ 即将进入计划模式？│
  └──┬──────────┬───┘
     │          │
  是 │          │ 否
     ▼          │
  ┌──────────┐  │
  │已头脑风暴？│  │
  └──┬───┬───┘  │
  否 │   │ 是   │
     ▼   │      │
  调用   │      │
  brainstorm     │
  技能   │      │
     │   │      │
     └───┘      │
         │      │
         ▼      ▼
  ┌──────────────────┐
  │ 可能有技能适用？  │ ← 哪怕只有 1%
  └──┬───────────┬──┘
  是 │           │ 否
     ▼           ▼
  调用 Skill    直接响应
  工具加载技能
     │
     ▼
  宣布："Using [skill] to [purpose]"
     │
     ▼
  技能有 Checklist？── 是 → 创建 TodoWrite 任务列表
     │
    否
     │
     ▼
  严格遵循技能指令
```

## 技能优先级：解决冲突

当多个技能可能同时适用时：

```
  ┌──────────────────────────────────────────────────┐
  │  优先级规则：                                     │
  │                                                    │
  │  1. 流程技能优先（brainstorming, debugging）       │
  │     → 它们决定 HOW to approach                     │
  │                                                    │
  │  2. 实现技能其次（subagent-driven, executing）     │
  │     → 它们指导 execution                           │
  │                                                    │
  │  例："Let's build X"                               │
  │  → brainstorming 先，然后实现技能                   │
  │                                                    │
  │  例："Fix this bug"                                │
  │  → debugging 先，然后领域特定技能                   │
  └──────────────────────────────────────────────────┘
```

## 技能类型：严格 vs 灵活

```
  严格型技能（Rigid）：
  ────────────────────
  - TDD、debugging、verification
  - 必须严格遵循，不要偏离纪律
  - "Violating the letter of the rules is violating
     the spirit of the rules."

  灵活型技能（Flexible）：
  ──────────────────────
  - patterns、explore mode
  - 将原则适配到上下文
  - 按情况调整

  技能本身会告诉你它是哪种类型。
```

## 宣布机制

技能加载后，AI 必须宣布：

```
  "I'm using the [skill-name] skill to [purpose]."
```

这看似简单，实则是**元认知控制**——迫使 AI 明确意识到自己正在使用哪个技能，防止"隐性跳过"（以为在用但实际没在用）。

### 各技能的宣布语

| 技能 | 宣布语 |
|------|--------|
| `writing-plans` | "I'm using the writing-plans skill to create the implementation plan." |
| `executing-plans` | "I'm using the executing-plans skill to implement this plan." |
| `subagent-driven-development` | 隐含在流程中（通过 Task 派遣体现） |
| `using-git-worktrees` | "I'm using the using-git-worktrees skill to set up an isolated workspace." |
| `finishing-a-development-branch` | "I'm using the finishing-a-development-branch skill to complete this work." |
