# 智能体激活机制

Superpowers 的智能体体系不是通过显式注册或类实例化激活的，而是通过**Hook 注入 + 技能调度 + Task 派遣**三层机制实现。

## 三层激活架构

```
  ┌──────────────────────────────────────────────────────────────┐
  │                     三层激活架构                                │
  │                                                              │
  │  Layer 1: SessionStart Hook                                  │
  │  ┌────────────────────────────────────────────────────────┐  │
  │  │  会话启动时自动注入 using-superpowers 技能              │  │
  │  │  → Controller 获得技能调度能力                          │  │
  │  │  → 所有后续技能调用都源于此                             │  │
  │  └────────────────────────────────────────────────────────┘  │
  │                         │                                    │
  │  Layer 2: Skill Dispatch                                    │
  │  ┌────────────────────────────────────────────────────────┐  │
  │  │  Controller 按技能指令决定何时派遣哪种子智能体          │  │
  │  │  → 构造 Prompt（从模板+上下文）                        │  │
  │  │  → 选择模型                                            │  │
  │  │  → 调用 Task 工具                                      │  │
  │  └────────────────────────────────────────────────────────┘  │
  │                         │                                    │
  │  Layer 3: Task Dispatch                                     │
  │  ┌────────────────────────────────────────────────────────┐  │
  │  │  平台的 Task 工具创建新的子智能体实例                    │  │
  │  │  → 隔离上下文                                          │  │
  │  │  → 执行任务                                            │  │
  │  │  → 返回结果给 Controller                               │  │
  │  └────────────────────────────────────────────────────────┘  │
  │                                                              │
  └──────────────────────────────────────────────────────────────┘
```

## Layer 1: SessionStart Hook

### 工作原理

```
  用户启动会话
       │
       ▼
  ┌──────────────────────────────┐
  │ hooks/hooks.json             │
  │                              │
  │ SessionStart 匹配器：        │
  │ "startup|clear|compact"      │
  │                              │
  │ 触发命令：                    │
  │ run-hook.cmd session-start   │
  └──────────────┬───────────────┘
                 │
                 ▼
  ┌──────────────────────────────────────┐
  │ hooks/session-start (bash 脚本)       │
  │                                      │
  │ 1. 检查旧版技能目录，构建警告          │
  │ 2. 读 using-superpowers/SKILL.md     │
  │ 3. JSON 转义                         │
  │ 4. 构造注入上下文：                    │
  │    <EXTREMELY_IMPORTANT>              │
  │    You have superpowers.              │
  │    [using-superpowers 完整内容]        │
  │    [工具映射（按平台）]                │
  │    </EXTREMELY_IMPORTANT>             │
  │ 5. 按平台输出不同 JSON 格式            │
  └──────────────────────────────────────┘
```

### 平台适配

Hook 脚本检测当前平台并输出对应的 JSON 格式：

| 平台 | 检测方式 | JSON 格式 |
|------|---------|----------|
| Cursor | `CURSOR_PLUGIN_ROOT` 环境变量 | `{ "additional_context": "..." }` |
| Claude Code | `CLAUDE_PLUGIN_ROOT`（无 `COPILOT_CLI`） | `{ "hookSpecificOutput": { "hookEventName": "SessionStart", "additionalContext": "..." } }` |
| Copilot CLI | `COPILOT_CLI=1` | `{ "additionalContext": "..." }` |
| OpenCode | 不走 Hook，走 plugin.js | `experimental.chat.messages.transform` |

### OpenCode 的特殊机制

OpenCode 不使用 SessionStart Hook，而是通过插件系统：

```
  .opencode/plugins/superpowers.js
       │
       ▼
  两个 Hook：
  ┌───────────────────────────────────────────┐
  │ config hook                               │
  │ → 注册 skills.paths 到 OpenCode 配置       │
  │ → OpenCode 自动发现所有 superpowers 技能   │
  └───────────────────────────────────────────┘
  ┌───────────────────────────────────────────┐
  │ experimental.chat.messages.transform hook  │
  │ → 在第一条用户消息前注入 bootstrap 上下文   │
  │ → 避免 system message token 膨胀           │
  │ → 避免多 system message 损坏某些模型       │
  └───────────────────────────────────────────┘
```

## Layer 2: Skill Dispatch

Controller 获得技能调度能力后，按以下规则决定何时调用哪个技能：

### 技能调度流程

```
  用户消息到达
       │
       ▼
  ┌─────────────────────┐
  │ 即将进入计划模式？    │
  └──┬──────────────┬───┘
     │              │
  是 │              │ 否
     ▼              │
  ┌─────────────┐   │
  │已头脑风暴？  │   │
  └──┬──────┬───┘   │
  否 │      │ 是     │
     ▼      ▼       │
  调用       │       │
  brainstorm │       │
  技能       │       │
     │       │       │
     └───────┘       │
             │       │
             ▼       ▼
  ┌─────────────────────┐
  │ 可能有技能适用？      │  ← 哪怕只有 1% 可能
  └──┬──────────────┬───┘
  是 │              │ 否（确定无疑）
     ▼              ▼
  调用 Skill       直接响应
  工具加载技能
     │
     ▼
  宣布："Using [skill] to [purpose]"
     │
     ▼
  技能有清单？── 是 → 创建 TodoWrite 任务列表
     │
    否
     │
     ▼
  严格遵循技能指令
```

### 技能优先级

```
  多个技能可能同时适用时的优先级：
  ────────────────────────────

  1. 流程技能优先（brainstorming, systematic-debugging）
     → 决定 HOW to approach

  2. 实现技能其次（subagent-driven-development, executing-plans）
     → 指导 execution

  例："Let's build X" → brainstorming → implementation skills
  例："Fix this bug"  → debugging → domain-specific skills
```

## Layer 3: Task Dispatch

Controller 通过 Task 工具创建子智能体：

### 派遣过程

```
  Controller 决定派遣子智能体
       │
       ▼
  ┌─────────────────────────────────┐
  │ 1. 选择模板                      │
  │    - implementer-prompt.md      │
  │    - spec-reviewer-prompt.md    │
  │    - code-quality-reviewer.md   │
  │    - code-reviewer.md           │
  └─────────────────────────────────┘
       │
       ▼
  ┌─────────────────────────────────┐
  │ 2. 填充模板                      │
  │    - 任务完整文本（粘贴，不让读） │
  │    - 场景上下文                  │
  │    - 工作目录                    │
  │    - git SHA（审阅者需要）       │
  └─────────────────────────────────┘
       │
       ▼
  ┌─────────────────────────────────┐
  │ 3. 选择模型                      │
  │    - 机械任务 → 便宜模型         │
  │    - 集成任务 → 标准模型         │
  │    - 审阅任务 → 最强模型         │
  └─────────────────────────────────┘
       │
       ▼
  ┌─────────────────────────────────┐
  │ 4. 调用 Task 工具                │
  │    Task tool (general-purpose): │
  │      description: "..."         │
  │      prompt: |                  │
  │        [填充后的模板]            │
  └─────────────────────────────────┘
       │
       ▼
  平台创建新的隔离子智能体实例
       │
       ▼
  子智能体执行 → 返回结果给 Controller
```

### 上下文隔离保证

```
  ┌──────────────────┐     ┌──────────────────┐
  │  Controller       │     │  子智能体          │
  │                  │     │                  │
  │  拥有：           │ ──▶ │  只获得：          │
  │  - 完整会话历史   │     │  - 精确任务文本   │
  │  - 所有技能知识   │     │  - 架构上下文     │
  │  - 前序任务结果   │     │  - 工作目录       │
  │  - 用户偏好       │     │                  │
  │                  │     │  绝不获得：        │
  │  不传递：         │     │  - Controller 历史 │
  │  - 会话历史       │     │  - 其他任务结果   │
  │  - 全局状态       │     │  - 用户偏好       │
  └──────────────────┘     └──────────────────┘

  为什么？
  1. 防止上下文污染——子智能体不会被无关信息干扰
  2. 节约 token——不需要传递完整历史
  3. 可重复——同样输入产生类似输出
  4. 可并行——子智能体之间不共享状态
```

## 持久化智能体的激活

`agents/code-reviewer.md` 的激活走不同路径：

```
  Claude Code：
  ────────────
  agents/ 目录扫描 → 解析 frontmatter → 注册 Agent
       │
       ├── 自动激活：匹配 description 中的触发条件
       │   例：用户说"I've finished step 3" → 平台匹配 → 自动调用
       │
       └── 显式调用：技能中使用 Task tool (superpowers:code-reviewer)
           例：subagent-driven-development 中指定使用此 Agent

  OpenCode：
  ─────────
  .opencode/plugins/superpowers.js → config hook → 注册 skills.paths
  → skill tool 可发现和加载
  → 技能内引用 code-reviewer Agent 时通过 Task tool 派遣
```

## 工具映射

不同平台的工具名称映射：

| Claude Code 工具 | OpenCode 等价物 | 用途 |
|-----------------|---------------|------|
| `Skill` | `skill` | 加载技能 |
| `TodoWrite` | `todowrite` | 任务追踪 |
| `Task`（子智能体） | `@mention` / Task 工具 | 派遣子智能体 |
| `Read` / `Write` / `Edit` | 原生工具 | 文件操作 |
| `Bash` | 原生工具 | 命令执行 |
