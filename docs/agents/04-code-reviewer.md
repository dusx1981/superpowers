# 代码审阅者（Code Reviewer）

代码审阅者在 Superpowers 中有**两种形态**：子智能体模板和持久化智能体。两者角色相同但激活方式不同。

## 形态一：子智能体模板（code-quality-reviewer）

在 `subagent-driven-development` 流程中作为**两阶段审阅的第二道关**。

```
  ┌─────────────────────────────────────────────────┐
  │         代码质量审阅者子智能体                     │
  │                                                  │
  │  角色：验证实现是否构建得好（干净、可测试、可维护）  │
  │  方法：基于 git diff (base_sha..head_sha) 审阅    │
  │  模板：skills/subagent-driven-development/       │
  │         code-quality-reviewer-prompt.md          │
  │  前置：spec-reviewer 必须已通过                    │
  │                                                  │
  │  生命周期：单次审阅，审完即销毁                     │
  └─────────────────────────────────────────────────┘
```

### 使用的技能

通过 `requesting-code-review` 技能激活，使用 `requesting-code-review/code-reviewer.md` 模板。

### 额外检查项

在标准代码质量关注之外，此审阅者还检查：

- 每个文件是否有单一清晰职责和定义良好的接口？
- 单元是否分解为可独立理解和测试的？
- 是否遵循计划中的文件结构？
- 是否创造了已经很大的新文件，或显著膨胀了现有文件？（不标记已有文件大小——关注此变更贡献了什么）

### 输出格式

```
  ### Strengths
  [具体列出做得好的地方]

  ### Issues
  #### Critical (Must Fix)
  [Bug、安全问题、数据丢失风险、功能损坏]
  #### Important (Should Fix)
  [架构问题、缺失功能、错误处理不当、测试缺口]
  #### Minor (Nice to Have)
  [代码风格、优化机会、文档改进]

  每个问题：file:line + 什么问题 + 为什么重要 + 怎么修

  ### Assessment
  Ready to merge? [Yes/No/With fixes]
  Reasoning: [1-2 句技术评估]
```

## 形态二：持久化智能体（agents/code-reviewer.md）

在 `agents/` 目录中定义，可被 Claude Code **原生发现和调用**。

```
  ┌─────────────────────────────────────────────────┐
  │         持久化代码审阅者                           │
  │                                                  │
  │  文件：agents/code-reviewer.md                   │
  │  模型：inherit（继承调用者模型）                   │
  │  触发：主要项目步骤完成后                          │
  │                                                  │
  │  与子智能体模板的区别：                            │
  │  - 由平台自动发现，不需要技能派遣                  │
  │  - 继承调用者模型（不单独选模型）                  │
  │  - 更通用（不限于 subagent-driven 流程）          │
  └─────────────────────────────────────────────────┘
```

### 六大审阅维度

```
  ┌───────────────────────────────────────────────────────┐
  │                                                       │
  │  1. 计划对齐分析                                       │
  │     ──────────────                                     │
  │     实现与原始计划/步骤描述对比                         │
  │     识别偏差：是合理改进还是问题偏离？                  │
  │     验证所有计划功能已实现                              │
  │                                                       │
  │  2. 代码质量评估                                       │
  │     ──────────────                                     │
  │     模式/约定、错误处理、类型安全                       │
  │     代码组织、命名、可维护性                            │
  │     测试覆盖和质量、安全/性能问题                      │
  │                                                       │
  │  3. 架构与设计审阅                                     │
  │     ──────────────                                     │
  │     SOLID 原则、关注点分离、松耦合                      │
  │     与现有系统集成、可扩展性                            │
  │                                                       │
  │  4. 文档与标准                                         │
  │     ──────────                                         │
  │     注释、函数文档、内联注释                            │
  │     项目特定编码标准                                    │
  │                                                       │
  │  5. 问题识别与建议                                     │
  │     ────────────────                                   │
  │     Critical / Important / Suggestions 三级分类        │
  │     每个问题：具体示例 + 可操作建议                    │
  │     计划偏差说明：是有问题还是有益的？                  │
  │                                                       │
  │  6. 通信协议                                           │
  │     ────────                                           │
  │     重大偏差 → 要求编码 Agent 确认                     │
  │     计划本身问题 → 建议更新计划                         │
  │     实现问题 → 提供清晰修复指导                         │
  │     总是先肯定做得好的，再指出问题                     │
  │                                                       │
  └───────────────────────────────────────────────────────┘
```

### 触发条件（来自 description 中的 examples）

- 用户完成了一个计划步骤："I've finished implementing the user authentication system as outlined in step 3"
- 用户完成了一个重要功能："The API endpoints for the task management system are now complete - that covers step 2"

## 两种形态的对比

| 维度 | 子智能体模板 | 持久化智能体 |
|------|------------|------------|
| 位置 | `skills/subagent-driven-development/` | `agents/code-reviewer.md` |
| 激活 | Controller 通过技能流程派遣 | 平台自动发现/用户触发 |
| 模型 | Controller 按复杂度选择 | inherit（继承调用者） |
| 上下文 | 精确构造（任务+diff+规范） | 更通用（计划+实现+需求） |
| 范围 | 单个任务 | 主要项目步骤 |
| 前置 | spec-reviewer 必须先通过 | 无前置 |
| 反馈 | Strengths/Issues/Assessment | 同左 + 通信协议 |

## 关键规则

```
  DO:
  - 按实际严重度分类（不是所有都是 Critical）
  - 具体（file:line，不模糊）
  - 解释为什么问题重要
  - 肯定优点
  - 给出明确判定

  DON'T:
  - 不检查就说"looks good"
  - 把小事标为 Critical
  - 审阅没读过的代码
  - 模糊反馈（"improve error handling"）
  - 不给明确判定
```
