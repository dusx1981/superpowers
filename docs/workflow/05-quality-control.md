# 05 质量控制

质量控制确保 AI 的产出**达到标准**——不是"差不多"，而是"没有已知问题"。

## 反模式列表：计划的负面清单

writing-plans 技能定义了一个明确的反模式列表——这些东西出现在计划中就是**计划失败**：

```
  绝不允许（计划失败）：
  ────────────────────
  ✗ "TBD"、"TODO"、"implement later"、"fill in details"
  ✗ "Add appropriate error handling"（没展示怎么加）
  ✗ "Write tests for the above"（没有实际测试代码）
  ✗ "Similar to Task N"（必须重复代码）
  ✗ 只描述做什么不展示怎么做
  ✗ 引用未在任何任务中定义的类型/函数/方法
```

### 为什么 "Similar to Task N" 是反模式？

```
  AI 可能想写：
  ────────────
  "Task 5: 实现删除功能
   与 Task 3 的添加功能类似"

  问题：实现者可能乱序读任务
  Task 5 的实现者还没读过 Task 3
  → 不知道"类似"是什么意思
  → 猜测实现 → 出错

  正确做法：重复完整的代码
  "Task 5: 实现删除功能
   - 创建 `src/lib/delete.ts`
   - export function deleteItem(id: string): void { ... }"
```

## 自审清单：内部质检

多个技能都有自审步骤——AI 必须在产出后**用新鲜眼光**检查自己的工作。

### writing-plans 的自审

```
  写完计划后：
  ────────────
  1. 规范覆盖：规范的每个需求是否都有任务对应？
     → 列出任何缺口

  2. 占位符扫描：搜索反模式
     → TBD / TODO / "implement later" / "fill in details"
     → 发现 → 修复

  3. 类型一致性：前面定义的函数签名/类型名在后面是否一致？
     → Task 3 的 clearLayers() vs Task 7 的 clearFullLayers()
     → 这是 bug，必须修复
```

### brainstorming 的规范自审

```
  写完设计文档后：
  ────────────────
  1. 占位符扫描：TBD、TODO、不完整章节、模糊需求
  2. 内部一致性：各章节是否矛盾？架构是否匹配功能描述？
  3. 范围检查：是否聚焦于单个实施计划？是否需要分解？
  4. 歧义检查：需求是否可以被两种方式解读？
```

### 实现者的自审

```
  提交前必须自审：
  ────────────────
  完整性：完全实现了规范？遗漏需求？边界情况？
  质量：这是最好的工作？命名清晰？代码干净？
  纪律：没有过度构建（YAGNI）？只做了要求的？
  测试：测试验证真实行为？遵循 TDD？测试全面？

  发现问题 → 立即修复后再报告
```

### plan-document-reviewer 的自审

writing-plans 技能还有一个**独立的计划文档审阅者**（通过子智能体派遣）：

```
  | 检查类别 | 查找内容 |
  |---------|---------|
  | 完整性 | TODOs、占位符、不完整任务、缺失步骤 |
  | 规范对齐 | 计划覆盖规范需求、无重大范围蔓延 |
  | 任务分解 | 任务边界清晰、步骤可操作 |
  | 可构建性 | 工程师能按此计划执行而不卡住吗？ |

  校准：只标记会在实施中造成真正问题的东西
  措辞/风格偏好和"锦上添花"建议不算
```

## Checklist → TodoWrite：外部可追踪

有清单的技能**必须**创建 TodoWrite 任务列表：

```
  brainstorming 技能加载
       │
       ▼
  发现 9 项 Checklist
       │
       ▼
  必须创建 TodoWrite：
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

### 为什么要 TodoWrite？

```
  没有 TodoWrite：
  ────────────────
  AI 在脑子里跟踪进度
  → 可能跳步（"我觉得已经问过问题了"）
  → 可能遗漏（"忘了要自审"）
  → 用户看不到进度
  → 无法验证完整性

  有 TodoWrite：
  ────────────
  每一步都有可见的状态
  → 必须标记 in_progress 才能开始
  → 必须标记 completed 才能继续
  → 用户可以看到进度
  → 可以验证每步是否完成
```

### TodoWrite 的规则

```
  ┌──────────────────────────────────────────────────┐
  │  TodoWrite 规则：                                 │
  │                                                    │
  │  - 每次只标记一个任务为 in_progress                │
  │  - 完成后立即标记 completed（不批量）              │
  │  - 当前任务完成前不开始下一个                      │
  │  - 取消不再需要的任务                              │
  └──────────────────────────────────────────────────┘
```

## 测试反模式列表

TDD 技能附带了一个独立的 `testing-anti-patterns.md` 文件，控制测试本身的质量：

```
  三条铁律：
  ────────
  1. 绝不测试 mock 行为
  2. 绝不在生产类中添加仅用于测试的方法
  3. 绝不在不理解依赖的情况下 mock
```

### 测试 mock 行为 vs 测试真实行为

```
  ✗ 坏测试（测试 mock）：
  test('renders sidebar', () => {
    render(<Page />);
    expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
  });
  → 验证的是 mock 存在，不是组件工作

  ✓ 好测试（测试真实行为）：
  test('retries failed operations 3 times', async () => {
    let attempts = 0;
    const operation = () => {
      attempts++;
      if (attempts < 3) throw new Error('fail');
      return 'success';
    };
    const result = await retryOperation(operation);
    expect(result).toBe('success');
    expect(attempts).toBe(3);
  });
  → 验证的是真实行为
```

## 实现者的状态报告：结构化质量信号

实现者完成后的报告不是自由文本，而是**结构化的**：

```
  报告格式：
  ────────
  - Status: DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT
  - 实现了什么（或尝试了什么）
  - 测试了什么 + 测试结果
  - 变更文件列表
  - 自审发现
  - 任何问题或疑虑
```

结构化报告的价值：
1. **Status 字段** — Controller 可以程序化地判断下一步（DONE → 审阅，BLOCKED → 升级）
2. **测试结果** — 必须提供具体数字（"5/5 passing"），不能说"测试应该过了"
3. **变更文件** — 可追踪，方便 diff 和审阅
4. **自审发现** — 实现者自我暴露问题，减少审阅者的工作量
