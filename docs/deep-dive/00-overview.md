# Superpowers 任务自动规划：全链路总览

Superpowers 的任务自动规划不是单一模块，而是一条从想法到代码的完整流水线，由 14 个 skill、OpenSpec 工件系统和子智能体架构协同完成。

## 七阶段流水线

```
┌──────────────────────────────────────────────────────────────────────┐
│                  Superpowers 任务自动规划全链路                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ① 想法    ② 探索     ③ 提案      ④ 规划     ⑤ 实施     ⑥ 质控  ⑦ 收尾 │
│  ┌─────┐  ┌─────┐  ┌──────┐  ┌──────┐  ┌──────┐  ┌─────┐  ┌───┐  │
│  │brain│─▶│expl │─▶│prop  │─▶│plan  │─▶│impl │─▶│rev  │─▶│fin│  │
│  │storm│  │ore  │  │ose   │  │write │  │exec │  │iew  │  │ish│  │
│  └─────┘  └─────┘  └──────┘  └──────┘  └──────┘  └─────┘  └───┘  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

| 阶段 | Skill / 命令 | 核心输出 | 详文 |
|------|-------------|---------|------|
| ① 头脑风暴 | `brainstorming` | design.md（设计文档） | [01-brainstorming.md](01-brainstorming.md) |
| ② 探索模式 | `/opsx-explore` | 思考结晶（非强制输出） | [02-explore-mode.md](02-explore-mode.md) |
| ③ 提案 | `/opsx-propose` | proposal.md + design.md + tasks.md | [03-openspec-proposal.md](03-openspec-proposal.md) |
| ④ 规划 | `writing-plans` | 实施计划（原子步骤列表） | [04-writing-plans.md](04-writing-plans.md) |
| ⑤ 实施 | `subagent-driven-development` / `executing-plans` | 代码+测试+提交 | [05-implementation.md](05-implementation.md) |
| ⑥ 质量控制 | spec-reviewer + code-quality-reviewer | 审阅通过 | [06-quality-control.md](06-quality-control.md) |
| ⑦ 收尾 | `finishing-a-development-branch` | 合并/PR/归档 | [07-finishing.md](07-finishing.md) |

## 横切关注点

贯穿所有阶段的支撑系统：

| 关注点 | Skill | 详文 |
|--------|-------|------|
| Git Worktree 隔离 | `using-git-worktrees` | [08-crosscutting.md](08-crosscutting.md) |
| TDD 铁律 | `test-driven-development` | [08-crosscutting.md](08-crosscutting.md) |
| 并行调度 | `dispatching-parallel-agents` | [08-crosscutting.md](08-crosscutting.md) |
| OpenSpec 生命周期 | CLI 工具 (`openspec`) | [03-openspec-proposal.md](03-openspec-proposal.md) |

## 核心设计哲学

| 设计原则 | 体现 |
|---------|------|
| **零上下文假设** | 计划写给"零上下文工程师"，每步都有完整代码和命令 |
| **原子化** | 每步 2-5 分钟，一个动作，可独立验证 |
| **TDD 铁律** | 没有失败测试 = 没有产品代码。先写代码？删掉重来 |
| **子智能体隔离** | 每个任务全新上下文，不继承会话历史，避免上下文污染 |
| **两阶段审阅** | 先验证"做对了事"（规范合规），再验证"把事做对"（代码质量），顺序不可颠倒 |
| **不信任原则** | spec-reviewer 明确被告知"不要信任实现者的报告"，必须读实际代码 |
| **成本优化** | 机械任务用便宜模型，架构/审阅用最强模型 |
| **强制升级** | 实现者卡住不强迫重试——换模型/拆任务/升级给人 |
| **YAGNI** | 从 brainstorming 到 plan 到 spec-reviewer，每一层都检查是否过度构建 |
| **可中断性** | 任何阶段遇到阻塞都可以停下问人，不猜 |
| **工件驱动** | OpenSpec 工件是有状态、可追踪、可恢复的，不是一次性文档 |
| **并行意识** | 独立问题可并行调度，但实现任务不并行（避免文件冲突） |

## 完整旅程示例

用户："我想给应用加暗黑模式"

```
① brainstorming
   "暗黑模式？CSS 变量切换？还是主题系统？用户偏好存哪？"
   → 提出 2-3 方案 → 用户选方案 → 分段确认设计
   → 写 design.md → 自审 → 用户审阅 → 批准

② using-git-worktrees
   git worktree add .worktrees/dark-mode -b feature/dark-mode
   npm install → npm test (47 passing)

③ writing-plans
   设计文件 → 原子化任务列表
   Task 1: CSS 变量基础设施 (5 steps x Red-Green-Refactor)
   Task 2: 主题切换组件 (5 steps)
   Task 3: 用户偏好持久化 (5 steps)
   Task 4: 系统偏好检测 (5 steps)
   → 自审（无占位符、类型一致、规范全覆盖）

④ subagent-driven-development
   Task 1: implementer → spec-reviewer ✅ → code-quality ✅
   Task 2: implementer → "Context 还是 Redux？" → "用 Context"
           → DONE_WITH_CONCERNS → spec-reviewer ❌ (缺少检测)
           → implementer 修复 → spec-reviewer ✅ → code-quality ❌ (魔法字符串)
           → implementer 修复 → code-quality ✅
   ... 所有任务完成 ...
   最终审阅子智能体 → ✅

⑤ finishing-a-development-branch
   npm test → 52 passing ✅
   → "1. 本地合并  2. 创建 PR  3. 保持  4. 丢弃"
   → 用户选 2 → git push → gh pr create → 清理 worktree
```
