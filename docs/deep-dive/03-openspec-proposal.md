# 阶段 ③：提案（/opsx-propose）

将想法**形式化为 OpenSpec 工件**。OpenSpec 是 superpowers 的变更管理系统。

## OpenSpec 工件体系

默认使用 `spec-driven` 模式（在 `openspec/config.yaml` 中配置）：

```
  spec-driven 模式的工件依赖图：

  ┌────────────┐     依赖       ┌────────────┐     依赖       ┌──────────┐
  │proposal.md │──────────────▶│ design.md  │──────────────▶│tasks.md  │
  │            │               │            │               │          │
  │ 做什么&为什么│               │ 怎么做      │               │ 实施步骤  │
  └────────────┘               └────────────┘               └──────────┘
       ▲                                                         │
       │                     applyRequires                        │
       └─────────────────────────────────────────────────────────┘
                         只有 tasks 完成后才能 apply

  辅助工件：
  ┌─────────────┐
  │specs/       │  每个 capability 一个 spec.md（delta spec 模式）
  │  <name>/    │  实现时变更的规范，归档时合并到主 specs
  │    spec.md  │
  └─────────────┘
```

## 工作流程

```
  ① openspec new change "<name>"
     → 创建 openspec/changes/<name>/ + .openspec.yaml
         ▼
  ② openspec status --change "<name>" --json
     → 解析 applyRequires（如 ["tasks"]）和 artifacts 依赖图
         ▼
  ③ 按依赖顺序创建工件（无依赖的先做）：
     for each ready artifact:
       a. openspec instructions <id> --change "<name>" --json
          → 获取 template / context / rules / instruction / outputPath
       b. 读已完成的依赖工件
       c. 按 template 填写，context/rules 作为约束（不写入文件）
       d. 重新运行 status 检查完成状态
         ▼
  ④ 所有 applyRequires 工件完成 → 可以 /opsx-apply
```

## 关键设计

### context 和 rules 是给 AI 的约束

`openspec instructions` 返回的 JSON 中：
- `context`：项目背景（约束给你，不写入输出）
- `rules`：工件特定规则（约束给你，不写入输出）
- `template`：输出文件的结构模板
- `instruction`：特定工件类型的指导

**绝不要**将 `<context>`、`<rules>`、`<project_context>` 块复制到工件文件中。

### 依赖感知的创建顺序

每个工件声明其依赖。只有依赖满足的工件才能被创建。这保证了：
- `proposal.md` 总是先创建（无依赖）
- `design.md` 依赖 `proposal.md`，所以第二个
- `tasks.md` 依赖 `design.md`，所以第三个

### config.yaml 示例

```yaml
schema: spec-driven

# 项目上下文（可选）
context: |
  Tech stack: TypeScript, React, Node.js
  We use conventional commits
  Domain: e-commerce platform

# 每个工件的规则（可选）
rules:
  proposal:
    - Keep proposals under 500 words
    - Always include a "Non-goals" section
  tasks:
    - Break tasks into chunks of max 2 hours
```

## OpenSpec 变更生命周期

```
  ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
  │   new     │──▶│  propose  │──▶│   apply   │──▶│  archive  │
  │           │    │           │    │           │    │           │
  │ 创建目录   │    │ 生成工件   │    │ 执行任务   │    │ 归档+同步  │
  │.openspec  │    │ proposal  │    │ 逐个实现   │    │ specs     │
  │.yaml      │    │ design    │    │ - [ ]→[x] │    │           │
  └───────────┘    │ tasks     │    └───────────┘    └───────────┘
                   └───────────┘
                        │
                   openspec CLI 驱动：
                   - status --json（依赖图+完成状态）
                   - instructions <id>（模板+约束+指导）
                   - list --json（所有变更）

  归档时 delta specs 合并到主 specs：
  openspec/changes/<name>/specs/  →  openspec/specs/<capability>/spec.md
```

## 守卫

- 创建所有 `applyRequires` 定义的实施所需工件
- 创建新工件前总是读依赖工件
- 如果上下文严重不清楚，问用户——但倾向做合理决策保持动力
- 如果同名变更已存在，问用户是继续还是新建
- 写入后验证每个工件文件存在
