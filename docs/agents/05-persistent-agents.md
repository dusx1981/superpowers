# 持久化智能体（Persistent Agents）

Superpowers 有两种智能体存在形式：**子智能体模板**（由技能在运行时动态创建）和**持久化智能体**（在 `agents/` 目录中静态定义，可被平台原生发现）。

## agents/ 目录

```
  agents/
  └── code-reviewer.md    ← 目前唯一的持久化智能体
```

## 持久化智能体 vs 子智能体模板

```
  ┌─────────────────────────────────────────────────────────────┐
  │                                                             │
  │  持久化智能体                        子智能体模板             │
  │  ─────────────                       ─────────────          │
  │                                                             │
  │  位置：agents/ 目录                  位置：skills/ 子目录    │
  │  格式：YAML frontmatter + body       格式：Prompt 模板 .md  │
  │  发现：平台自动                       发现：技能指令          │
  │  生命周期：永久存在                   生命周期：单次派遣销毁  │
  │  上下文：通用                         上下文：精确构造        │
  │  模型：可指定（如 inherit）           模型：Controller 选择   │
  │                                                             │
  └─────────────────────────────────────────────────────────────┘
```

## code-reviewer 的 YAML Frontmatter

```yaml
---
name: code-reviewer
description: |
  Use this agent when a major project step has been completed
  and needs to be reviewed against the original plan and coding
  standards. Examples:
    <example>Context: ...
      user: "I've finished implementing step 3"
      assistant: "Let me use the code-reviewer agent..."
    </example>
model: inherit
---
```

关键字段：

| 字段 | 说明 |
|------|------|
| `name` | 智能体标识符，用于技能中引用（如 `Task tool (superpowers:code-reviewer)`） |
| `description` | 触发条件描述 + 示例，平台据此决定何时自动激活 |
| `model` | 使用的模型。`inherit` 表示继承调用者的模型 |

## 发现机制

不同平台对持久化智能体的发现方式：

### Claude Code

```
  agents/ 目录中的 .md 文件
       │
       ▼
  Claude Code 扫描 agents/ 目录
       │
       ▼
  解析 YAML frontmatter
       │
       ▼
  注册为可用 Agent
       │
       ▼
  匹配 description 中的触发条件
       │
       ▼
  自动激活或用户/技能显式调用
```

### OpenCode

```
  .opencode/plugins/superpowers.js
       │
       ▼
  config hook → 注册 skills.paths
       │
       ▼
  skills/ 目录中的 SKILL.md
       │
       ▼
  skill tool 可列出和加载
       │
       ▼
  技能内引用 agents/ → Task tool 派遣
```

### Codex

```
  ~/.agents/skills/ 目录
       │
       ▼
  Codex 原生扫描 → 解析 SKILL.md frontmatter
       │
       ▼
  按需激活
```

## 为什么目前只有 code-reviewer 是持久化智能体？

持久化智能体适合**跨技能、跨场景通用**的角色。code-reviewer 之所以是持久化智能体，因为：

1. **多入口调用** — 被 `subagent-driven-development`、`requesting-code-review`、用户直接请求等多种方式使用
2. **独立身份** — 不依赖特定技能流程，可以独立激活
3. **通用审阅** — 不限于某个计划或任务，适用于任何代码审阅场景

其他智能体（实现者、spec-reviewer）是**特定技能流程的组件**，只在该技能上下文中有意义，因此作为技能内的模板更合适。

## 潜在的扩展方向

其他可能适合持久化智能体化的角色：

| 角色 | 当前形态 | 持久化价值 |
|------|---------|-----------|
| spec-reviewer | 子智能体模板 | 中 — 可被 brainstorming 和 writing-plans 复用 |
| implementer | 子智能体模板 | 低 — 太依赖技能上下文 |
| plan-reviewer | 子智能体模板 | 中 — 可被 writing-plans 和 openspec-propose 复用 |
| debugger | 内嵌在 systematic-debugging 技能中 | 高 — 跨场景通用 |
