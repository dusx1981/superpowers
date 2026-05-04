# 10 综合案例：暗黑模式全链路追踪

用一个真实场景走完全链路，展示每一步控制如何生效。

## 场景

用户在 OpenCode 中打开项目，输入："帮我加一个暗黑模式"

---

## Step 0：会话启动 — 注入控制

用户还没说任何话，控制已经开始。

```
  用户打开 OpenCode 会话
       │
       ▼  .opencode/plugins/superpowers.js
       │
       ▼  config hook → 注册 skills/ 目录到 OpenCode
       │
       ▼  messages.transform hook → 在首条用户消息前注入：

  <EXTREMELY_IMPORTANT>
  你拥有超能力。

  **重要：using-superpowers 技能内容已包含在下方。
  它已经加载——你当前正在遵循它。
  不要使用 skill 工具再次加载 "using-superpowers"——那是冗余的。**

  [技能完整内容]

  **OpenCode 工具映射：**
  - `TodoWrite` → `todowrite`
  - `Task` → @mention
  - `Skill` → `skill`
  - `Read`/`Write`/`Edit`/`Bash` → 原生工具
  </EXTREMELY_IMPORTANT>
```

**控制生效点**：`<EXTREMELY_IMPORTANT>` 标签 + "你已经加载" 声明 → AI 在看到第一个用户消息时已经处于"必须使用技能"模式。

---

## Step 1：用户消息到达 — 触发控制

```
  用户："帮我加一个暗黑模式"
       │
       ▼  AI 的思考过程（被注入控制引导）：

  "这是创造性工作吗？" → 是，加暗黑模式是创建功能
  "brainstorming 技能适用吗？" → 是
    → description: "You MUST use this before any creative work"
    → 哪怕只有 1% 可能也必须调用
       │
       ▼  AI 调用 skill 工具加载 brainstorming
       │
       ▼  宣布："Using brainstorming to explore dark mode requirements"
       │
       ▼  brainstorming 技能加载 → 发现 9 项 Checklist
       │
       ▼  创建 TodoWrite：
  - [ ] 1. 探索项目上下文
  - [ ] 2. 提供视觉伴侣
  - [ ] 3. 逐一提问澄清
  - [ ] 4. 提出 2-3 方案
  - [ ] 5. 分段呈现设计
  - [ ] 6. 写设计文档
  - [ ] 7. 规范自审
  - [ ] 8. 用户审阅规范
  - [ ] 9. 转入写计划
```

**控制生效点**：
1. 1% 规则 → AI 不能说"这太简单不需要 brainstorming"
2. description 字段 → 自动匹配"创造性工作"
3. Checklist → TodoWrite → AI 不能跳步，每步都必须标记

---

## Step 2：brainstorming 过程 — 门禁控制

```
  AI 想直接写代码？
       │
       ▼  HARD-GATE 触发：
  "在呈现设计并获得用户批准之前，
   不得调用任何实现技能、写任何代码..."
       │
       ▼  AI 遵守：先问问题

  AI："暗黑模式需要满足什么场景？
       1. CSS 变量切换（简单）
       2. 完整主题系统（可扩展）
       3. 跟随系统偏好自动切换
       你倾向哪种？"

  用户："CSS 变量切换 + 跟随系统偏好"

  AI 呈现设计分段：
  "架构：用 CSS 自定义属性定义明暗两套色板，
   通过 data-theme 属性切换，
   matchMedia 检测系统偏好。
   这部分看起来对吗？"

  用户："对"

  AI 继续下一段...

  用户批准整体设计
       │
       ▼  写设计文档 → 自审 → 用户审阅文档
       │
       ▼  唯一出口控制：
  "终态是调用 writing-plans。
   不要调用任何其他实现技能。"
       │
       ▼  AI 调用 writing-plans 技能
```

**控制生效点**：
1. HARD-GATE → 阻止直接写代码
2. Anti-Pattern 专节 → 阻止"这太简单了"合理化
3. 唯一出口 → 只能去 writing-plans，不能跳到 TDD 或 subagent-driven
4. 用户审阅门 → 两道门（设计批准 + 文档审阅）

---

## Step 3：写计划 — 质量控制

```
  AI 使用 writing-plans 技能写计划
       │
       ▼  宣布："I'm using the writing-plans skill
            to create the implementation plan."
       │
       ▼  写原子化步骤：

  Task 1: CSS 变量基础设施
  - [ ] Step 1: 写失败测试
  - [ ] Step 2: 运行确认失败
  - [ ] Step 3: 写最小实现
  - [ ] Step 4: 运行确认通过
  - [ ] Step 5: 提交

  Task 2: 主题切换组件
  ...

       │
       ▼  自审清单触发：
  1. 规范覆盖：暗黑模式所有需求都有任务？
     → 缺少 "系统偏好检测" → 添加 Task 4
  2. 占位符扫描：搜索反模式
     → 无 TBD/TODO → 通过
  3. 类型一致性：
     → Task 3 的 toggleTheme() vs Task 4 的 switchTheme()
     → 不一致！统一为 toggleTheme()
       │
       ▼  提供执行选项：
  "1. Subagent-Driven（推荐）
   2. Inline Execution
   你选哪个？"
```

**控制生效点**：
1. 反模式列表 → 阻止写 "TODO: add tests"
2. 自审清单 → 发现遗漏任务和类型不一致
3. 原子化步骤 → 每步 2-5 分钟，不可跳步

---

## Step 4：执行 — 顺序控制

```
  用户选择 Subagent-Driven
       │
       ▼  AI 加载 subagent-driven-development 技能
       │
       ▼  流程图规定了严格顺序：

  ═══ Task 1: CSS 变量基础设施 ═══

  ┌─ 派遣实现者子智能体 ──────────────────────────────┐
  │  Prompt 包含：                                     │
  │  - Task 1 完整文本（粘贴，不让子智能体读文件）     │
  │  - 上下文：Svelte + Vite 项目，CSS 自定义属性      │
  │  - 工作目录：.worktrees/dark-mode                  │
  │  - "如果你有问题，现在就问"                        │
  │  - "遵循 TDD Red-Green-Refactor"                  │
  │  - "完成后自审，报告 Status"                       │
  └────────────────────────────────────────────────────┘
       │
       ▼  实现者执行 TDD 循环：
  RED:   写 test_theme_variables_exist() → 失败 ✓
  GREEN: 写 CSS 变量定义 → 通过 ✓
  REFACTOR: 整理变量命名 → 通过 ✓
  提交："feat: add CSS custom properties for theming"
  自审：通过
  报告：DONE, 5/5 测试通过

       │
       ▼  派遣规范审阅者子智能体
  ┌────────────────────────────────────────────────────┐
  │  Prompt 包含：                                     │
  │  - Task 1 的完整需求                               │
  │  - 实现者的报告                                    │
  │  - "CRITICAL: Do Not Trust the Report"             │
  │  - "读实际代码，逐行比对需求"                      │
  └────────────────────────────────────────────────────┘
       │
       ▼  规范审阅者读代码 → ✅ Spec compliant

       │
       ▼  派遣代码质量审阅者子智能体
  ┌────────────────────────────────────────────────────┐
  │  Prompt 包含：                                     │
  │  - git diff (base..head)                           │
  │  - 实现者的报告                                    │
  │  - 额外检查：文件职责、可测试性、文件结构          │
  └────────────────────────────────────────────────────┘
       │
       ▼  代码质量审阅者 → ✅ Approved

       │
       ▼  标记 Task 1 完成
```

**控制生效点**：
1. 流程图 → 规范审阅必须在代码质量审阅之前
2. 不信任原则 → 审阅者读实际代码，不信任实现者报告
3. 全新子智能体 → 审阅者不知道实现者怎么想的
4. TDD 控制 → 实现者必须先写失败测试

---

## Step 5：审阅不通过时 — 失败控制

```
  ═══ Task 2: 主题切换组件 ═══

  实现者报告：DONE_WITH_CONCERNS
  "完成了，但我不确定 onMount 中的 matchMedia
   监听器会不会在 SSR 时报错"

       │
       ▼  Controller 先读疑虑：
  "这是关于正确性的疑虑 → 先解决再审阅"

       │
       ▼  派遣规范审阅者

  规范审阅者报告：❌ Issues found:
  - Missing: 没有实现 onMount 内的 cleanup（需求说"防止内存泄漏"）
  - Extra: 加了一个 --transition-speed 变量（规范没要求）

       │
       ▼  实现者修复：
  - 添加 cleanup 逻辑
  - 移除 --transition-speed 变量

       │
       ▼  重新派遣规范审阅者（必须重新审阅！）
  规范审阅者 → ✅ Spec compliant

       │
       ▼  派遣代码质量审阅者
  代码质量审阅者 → ❌ Issues (Important):
  - 魔法字符串 "dark" 出现 3 次，应提取常量

       │
       ▼  实现者修复：
  - 提取 THEME_DARK 常量

       │
       ▼  重新派遣代码质量审阅者
  代码质量审阅者 → ✅ Approved

       │
       ▼  标记 Task 2 完成
```

**控制生效点**：
1. DONE_WITH_CONCERNS → Controller 先读疑虑再决定
2. 规范审阅发现遗漏和额外 → 必须修复后重新审阅
3. 代码审阅发现魔法字符串 → 必须修复后重新审阅
4. 审阅循环 → 不通过 = 修复 + 重新审阅，不能"看起来修了就过"

---

## Step 6：实现者卡住 — 升级链

```
  ═══ Task 3: localStorage 持久化 ═══

  实现者报告：BLOCKED
  "Svelte 的 store 订阅和 localStorage 的
   异步行为冲突，我不确定怎么处理"

       │
       ▼  Controller 评估：
  "绝不强迫同一模型无条件重试。
   实现者说卡住了，必须改变什么。"

  选项：
  1. 提供更多上下文（Svelte onMount + 生命周期）
  2. 换更强模型
  3. 拆分任务

       │
       ▼  Controller 选择：提供更多上下文 + 拆分任务
  "Svelte 中，store 的 subscribe 是同步的，
   但 localStorage.getItem 是同步 API，
   只需注意 onMount 中初始化即可。
   我把此任务拆为：3a 同步版 + 3b onMount 集成"

       │
       ▼  重新派遣实现者（同一模型，更多上下文 + 更小任务）
  实现者 → 完成 → DONE
```

**控制生效点**：
1. 四种状态 → BLOCKED 触发升级链
2. 禁止无条件重试 → 必须改变条件
3. 升级选项 → 更多上下文/更强模型/更小任务/升级给人

---

## Step 7：完成前 — 验证控制

```
  所有 4 个任务完成 ✅

  AI 想说："Great! 暗黑模式完成了！"
       │
       ▼  verification-before-completion 触发：
  "没有新鲜验证证据的完成声称 = 撒谎"

       │
       ▼  AI 必须先运行验证：

  $ npm test
  → 52/52 passing ✅

  $ npm run build
  → exit 0 ✅

       │
       ▼  现在才能声称：
  "所有测试通过（52/52），构建成功（exit 0）。
   暗黑模式实现完成。"
```

**控制生效点**：
1. 铁律 → 没运行验证 = 撒谎
2. "新鲜"要求 → 必须在本条消息中运行
3. 完整输出 → 必须读输出、检查退出码、数失败数

---

## Step 8：收尾 — 顺序控制的最后一环

```
       │
       ▼  finishing-a-development-branch 技能

  1. 验证测试（必须通过才继续）
     $ npm test → 52 passing ✅

  2. 确定基准分支 (main)

  3. 呈现 4 个选项：
     "1. 本地合并回 main
      2. 推送并创建 PR
      3. 保持分支现状
      4. 丢弃"

  用户选 2 → git push -u → gh pr create

  4. 清理 worktree
```

**控制生效点**：
1. 再次验证测试 → 合并前确认
2. 结构化选项 → 不问开放式"你想怎么办"
3. 清理 worktree → 不留垃圾

---

## 全链路控制回顾

```
  Step 0: 注入控制    → <EXTREMELY_IMPORTANT> 抢先植入技能意识
  Step 1: 触发控制    → 1% 规则 + description 匹配 → 必须用 brainstorming
  Step 2: 门禁控制    → HARD-GATE → 不得写代码；唯一出口 → 只能去 writing-plans
  Step 3: 质量控制    → 反模式列表 + 自审 → 发现遗漏任务和类型不一致
  Step 4: 顺序控制    → 流程图 → 规范审阅 → 代码审阅，顺序锁死
  Step 5: 信任控制    → 不信任原则 → 审阅者读实际代码
  Step 6: 失败控制    → BLOCKED → 升级链 → 提供上下文 + 拆任务
  Step 7: 验证控制    → 铁律 → 必须运行验证才能声称完成
  Step 8: 收尾控制    → 再次验证 + 结构化选项 + 清理
```
