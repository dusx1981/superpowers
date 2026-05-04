# 03 门禁控制

门禁控制是 Superpowers 中**最硬性的控制**——用 XML 标签和"绝不"语言在关键节点设置不可逾越的门槛。

## HARD-GATE：brainstorming 中的绝对门禁

### 机制

brainstorming 技能中有一个 XML 标签形式的硬门禁：

```
  <HARD-GATE>
  Do NOT invoke any implementation skill, write any code,
  scaffold any project, or take any implementation action
  until you have presented a design and the user has approved it.
  This applies to EVERY project regardless of perceived simplicity.
  </HARD-GATE>
```

### 为什么需要 HARD-GATE？

没有门禁时，AI 最常见的偷懒路径：

```
  用户："帮我加暗黑模式"
       │
       ▼
  AI 的判断路径（无 HARD-GATE）：
  ┌─────────────────────────────────────────┐
  │ "暗黑模式很简单，就是 CSS 变量切换"     │
  │ "我直接写几行代码就行"                  │
  │ "不需要设计文档"                        │
  └──────────────┬──────────────────────────┘
                 ▼
  直接写代码 → 缺少设计 → 返工

  AI 的判断路径（有 HARD-GATE）：
  ┌─────────────────────────────────────────┐
  │ "暗黑模式很简单，我直接..."             │
  │ "等等，HARD-GATE 说无论多简单都必须     │
  │  先呈现设计并获得用户批准"               │
  └──────────────┬──────────────────────────┘
                 ▼
  先问问题 → 提方案 → 呈现设计 → 等批准 → 才能实现
```

### "This Is Too Simple" 反模式

HARD-GATE 之后紧跟一个专门的反模式警告：

```
  ## Anti-Pattern: "This Is Too Simple To Need A Design"

  Every project goes through this process. A todo list,
  a single-function utility, a config change — all of them.
  "Simple" projects are where unexamined assumptions cause
  the most wasted work.
```

这针对的是 AI 最常见的合理化借口——"太简单了不需要设计"。

## 唯一出口控制

brainstorming 技能不仅控制入口（不能直接写代码），还控制**出口**（只能去哪里）：

```
  ┌──────────────────────────────────────────────────────────┐
  │  brainstorming 技能的出口声明：                           │
  │                                                          │
  │  "终态是调用 writing-plans。                              │
  │   不要调用 frontend-design、mcp-builder                   │
  │   或任何其他实现技能。                                    │
  │   brainstorming 之后你唯一调用的技能是 writing-plans。"   │
  └──────────────────────────────────────────────────────────┘
```

### 出口控制的价值

```
  没有出口控制：
  ──────────────
  brainstorming 完成 → AI 自由选择下一步
  → 可能跳到 subagent-driven-development（跳过了计划）
  → 可能跳到 TDD（跳过了计划和分解）
  → 结果：步骤缺失，质量下降

  有出口控制：
  ────────────
  brainstorming 完成 → 唯一下一步：writing-plans
  → 写计划 → 计划规定下一步
  → 结果：步骤完整，质量可控
```

## 用户审阅门

brainstorming 中有两个用户审阅门：

```
  审阅门 1：设计呈现
  ─────────────────
  "按架构/组件/数据流/错误处理/测试分段呈现设计，
   每段确认后才继续"

  审阅门 2：规范文档审阅
  ─────────────────────
  "规范已写入并提交到 <path>。请审阅并告知在开始
   编写实施计划前是否需要修改。"

  流程：
  呈现设计 → 用户批准 → 写文档 → 自审 → 用户审阅文档 → 用户批准 → writing-plans
       ↑           │                              ↑          │
       └── 不批准：修改设计 ──────────────────────┘  不批准：修改文档
```

## 红旗规则：各技能中的"绝不"清单

每个技能都有自己的红旗规则，用"绝不"（Never）语言设定不可逾越的底线：

### brainstorming 的红旗

| 规则 | 控制什么 |
|------|---------|
| 不调用实现技能 | 防止跳过设计 |
| 不写代码 | 防止跳过设计 |
| 不搭脚手架 | 防止跳过设计 |
| 范围太大先分解 | 防止一次做太多 |

### subagent-driven-development 的红旗

| 规则 | 控制什么 |
|------|---------|
| 绝不在 main/master 上开始实现 | 防止直接在主分支操作 |
| 绝不跳过审阅 | 防止跳过质量门控 |
| 绝不并行派遣多个实现子智能体 | 防止文件冲突 |
| 绝不跳过重新审阅 | 防止"看起来修了就过" |
| 规范审阅必须先于代码审阅 | 防止顺序颠倒 |

### TDD 的红旗

| 规则 | 控制什么 |
|------|---------|
| 先写代码？删掉重来 | 防止跳过测试先行 |
| 不留作"参考" | 防止合理化保留错误代码 |
| 不"适配" | 防止基于错误代码写测试 |

### verification-before-completion 的红旗

| 规则 | 控制什么 |
|------|---------|
| 没运行验证命令 = 撒谎 | 防止凭感觉声称完成 |
| 不说"应该没问题" | 防止乐观偏差 |
| 不信任 Agent 成功报告 | 防止被子智能体误导 |

## "绝不"语言的强度设计

Superpowers 使用了**语言强度梯度**来控制 AI 的服从程度：

```
  强度递增：
  ────────

  "Consider..."              ← 建议（低）— AI 可以忽略
  "Prefer..."                ← 偏好（中低）— AI 应该遵循但可以例外
  "Should..."                ← 应该（中）— AI 除非有强烈理由否则遵循
  "Must..." / "Always..."    ← 必须（高）— AI 不应该违背
  "Never..." / "Do NOT..."   ← 绝不（最高）— AI 绝不能违背
  "<HARD-GATE>..."           ← 门禁（绝对）— XML 标签 + 绝不 = 不可逾越
  "Iron Law" / "铁律"        ← 铁律（终极）— 违反 = 行为失败
```

### 铁律示例

TDD 技能中的铁律：

```
  NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

verification 技能中的铁律：

```
  NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

systematic-debugging 技能中的铁律：

```
  NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
```

铁律的格式特征：**全大写 + 否定句式 + 独立成行**。这种格式在 LLM 上下文中极度醒目，几乎不可能被忽略。
