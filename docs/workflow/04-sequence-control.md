# 04 顺序控制

顺序控制确保 AI **按正确顺序**执行步骤——不能跳步、不能倒序、不能并行会冲突的操作。

## 核心工具：流程图

Superpowers 大量使用 Graphviz `dot` 流程图来规定执行顺序。流程图不是装饰——它是**控制规范**。

### brainstorming 的流程图

```
  digraph brainstorming {
      "Explore project context" -> "Visual questions ahead?";
      "Visual questions ahead?" -> "Offer Visual Companion" [label="yes"];
      "Visual questions ahead?" -> "Ask clarifying questions" [label="no"];
      "Offer Visual Companion" -> "Ask clarifying questions";
      "Ask clarifying questions" -> "Propose 2-3 approaches";
      "Propose 2-3 approaches" -> "Present design sections";
      "Present design sections" -> "User approves design?";
      "User approves design?" -> "Present design sections" [label="no, revise"];
      "User approves design?" -> "Write design doc" [label="yes"];
      "Write design doc" -> "Spec self-review";
      "Spec self-review" -> "User reviews spec?";
      "User reviews spec?" -> "Write design doc" [label="changes requested"];
      "User reviews spec?" -> "Invoke writing-plans skill" [label="approved"];
  }
```

流程图规定了：
1. **唯一入口**：必须从 "Explore project context" 开始
2. **唯一出口**：必须到 "Invoke writing-plans skill" 结束
3. **分支条件**：每个菱形节点有明确的 yes/no 路径
4. **回环路径**：用户不批准时回到前面修改，不允许跳过

### subagent-driven-development 的流程图

```
  digraph process {
      rankdir=TB;

      subgraph cluster_per_task {
          "Dispatch implementer" -> "Implementer asks questions?";
          "Implementer asks questions?" -> "Answer questions" [label="yes"];
          "Answer questions" -> "Dispatch implementer";  // 重新派遣
          "Implementer asks questions?" -> "Implementer works" [label="no"];
          "Implementer works" -> "Dispatch spec reviewer";
          "Dispatch spec reviewer" -> "Spec matches?";
          "Spec matches?" -> "Implementer fixes spec gaps" [label="no"];
          "Implementer fixes spec gaps" -> "Dispatch spec reviewer";  // 重新审阅
          "Spec matches?" -> "Dispatch code quality reviewer" [label="yes"];
          "Dispatch code quality reviewer" -> "Quality approves?";
          "Quality approves?" -> "Implementer fixes quality issues" [label="no"];
          "Implementer fixes quality issues" -> "Dispatch code quality reviewer";  // 重新审阅
          "Quality approves?" -> "Mark task complete" [label="yes"];
      }
  }
```

## 审阅链：顺序不可颠倒

最关键的顺序控制是**审阅链**：

```
  ┌──────────┐     ┌──────────────┐     ┌──────────────┐
  │ 实现者   │────▶│ 规范合规审阅 │────▶│ 代码质量审阅 │
  │Implementer│    │Spec Reviewer │    │Code Reviewer │
  └──────────┘     └──────────────┘     └──────────────┘
       │                  │                     │
       │              不通过？               不通过？
       │                  │                     │
       │                  ▼                     ▼
       │◀─────────── 实现者修复 ◀───────── 实现者修复
       │                  │                     │
       │              重新审阅               重新审阅
       │                  │                     │
       │               通过                    通过
       │                  ▼                     ▼
       │             ✅ 规范合规            ✅ 代码质量
       │                                       │
       └───────────────────────────────────────┘
                            │
                            ▼
                      标记任务完成
```

### 为什么顺序不能颠倒？

```
  ✗ 错误顺序：代码质量审阅 → 规范合规审阅
  ────────────────────────────────────────────
  代码写得再漂亮，做错了事也没用
  例：完美实现了用户注册，但规范要求的是用户登录
  代码质量审阅会通过（代码好），但做错了事

  ✓ 正确顺序：规范合规审阅 → 代码质量审阅
  ────────────────────────────────────────────
  先确认"做对了事"，再确认"把事做对"
  例：先确认实现了用户登录 → 再检查代码质量
```

技能中的红旗规则强化了这一点：

```
  绝不：
  - 在规范合规通过前开始代码质量审阅（顺序不可颠倒）
  - 跳过审阅（规范合规或代码质量）
  - 在任一审阅有开放问题时移到下一个任务
```

## 审阅循环：不通过必须重审

审阅不是一次性的——发现问题时必须修复后重新审阅：

```
  审阅发现问题
       │
       ▼
  实现者修复（同一个子智能体实例）
       │
       ▼
  审阅者重新审阅  ←── 关键：必须重新审阅！
       │               不能"看起来修了就过"
    通过？── 否 → 回到修复
       │
      是
       ▼
    下一步
```

### 为什么不能"看起来修了就过"？

```
  AI 的倾向（无控制）：
  ────────────────────
  审阅者："第 45 行缺少错误处理"
  实现者修复
  Controller（不重审）："应该修好了，继续吧"
  → 可能修了但不完整、可能引入新问题、可能修了错误的地方

  AI 的行为（有控制）：
  ────────────────────
  审阅者："第 45 行缺少错误处理"
  实现者修复
  Controller → 重新派遣审阅者审阅
  审阅者读代码验证 → 确认修复 → 通过
  → 确保修复真正生效
```

## TDD 的 Red-Green-Refactor 顺序

TDD 技能控制了实现代码的最小粒度执行顺序：

```
  ┌────────┐     ┌────────┐     ┌──────────┐
  │  RED   │────▶│ GREEN  │────▶│ REFACTOR │──┐
  │写失败测试│     │最小实现  │     │ 清理     │  │
  └────────┘     └────────┘     └──────────┘  │
       ▲              │               │        │
       │              ▼               ▼        │
       │         验证通过          验证仍通过     │
       │              │               │        │
       │              └───────────────┘        │
       │                                       │
       └───────────────────────────────────────┘
                    下一个测试
```

顺序控制：

| 步骤 | 必须先做 | 绝不能做 |
|------|---------|---------|
| RED | 无（第一步） | 不看已有实现写测试 |
| GREEN | RED 失败验证 | 不加功能、不重构、不"改进" |
| REFACTOR | GREEN 通过 | 不加行为、不破坏测试 |
| 下一个测试 | REFACTOR 完成 | 不跳过红绿重构循环 |

### "先写代码？删掉"的控制

```
  TDD 铁律：
  ────────
  NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST

  先写代码？删除它。从头来。
  - 不留作"参考"
  - 不"适配"它写测试
  - 不看它
  - 删除意味着删除

  从测试重新实现。完事。
```

## systematic-debugging 的四阶段顺序

```
  Phase 1: 根因调查          ← 必须先完成
       │
       ▼
  Phase 2: 模式分析          ← 必须在 Phase 1 后
       │
       ▼
  Phase 3: 假设与测试        ← 必须在 Phase 2 后
       │
       ▼
  Phase 4: 实现              ← 必须在 Phase 3 后

  铁律：NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
  如果没完成 Phase 1，不能提出修复。
```

### 3+ 修复失败 → 质疑架构

```
  修复 1 失败 → 回到 Phase 1 重新分析
  修复 2 失败 → 回到 Phase 1 重新分析
  修复 3 失败 → 停下来！质疑架构
       │
       ▼
  "这是不是根本性的架构问题？"
  "我们是不是在用惯性坚持一个错误的模式？"
  → 与用户讨论，不再尝试更多修复
```

## 全链路的顺序依赖图

```
  SessionStart 注入
       │
       ▼
  using-superpowers 激活
       │
       ▼
  用户消息 → 技能调度
       │
       ├── 创造性工作 → brainstorming
       │       │
       │       ├── HARD-GATE：不得实现
       │       ├── 提问 → 提方案 → 呈现设计
       │       ├── 用户批准
       │       ├── 写文档 → 自审 → 用户审阅
       │       └── 唯一出口：writing-plans
       │               │
       │               ├── 反模式检查
       │               ├── 自审
       │               └── 提供执行选项
       │                       │
       │                       └── subagent-driven-development
       │                               │
       │                               ├── 每任务：实现者 → 规范审阅 → 代码审阅
       │                               └── 全部完成 → 最终审阅
       │                                       │
       │                                       └── finishing-a-development-branch
       │
       ├── Bug → systematic-debugging
       │       │
       │       ├── 根因调查 → 模式分析 → 假设 → 实现
       │       └── 3+ 失败 → 质疑架构
       │
       └── 声称完成 → verification-before-completion
               │
               └── 运行验证 → 确认 → 才能声称
```
