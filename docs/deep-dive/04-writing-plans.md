# 阶段 ④：规划（writing-plans）

把规范**拆解为可执行的原子步骤**。计划写给一个零上下文、品味可疑的工程师——他需要知道碰哪些文件、写什么代码、怎么测试、怎么验证。

## 核心原则

> "假设工程师是熟练的，但对我们的工具集和问题领域几乎一无所知。假设他们不太懂好的测试设计。"

## 任务粒度

每一步是一个动作（2-5 分钟）：

| 步骤 | 说明 |
|------|------|
| "写失败的测试" | 一步 |
| "运行确认失败" | 一步 |
| "写最小实现" | 一步 |
| "运行确认通过" | 一步 |
| "提交" | 一步 |

## 计划文档结构

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development
  (recommended) or superpowers:executing-plans to implement this plan.

**Goal:** [一句话描述]
**Architecture:** [2-3 句方法]
**Tech Stack:** [关键技术/库]

---

### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

## 文件结构

在定义任务之前，先映射哪些文件将被创建或修改，每个文件负责什么：

- 设计有清晰边界和定义良好接口的单元
- 一起变化的文件应该放在一起——按职责分割，不按技术层分割
- 在现有代码库中遵循已有模式
- 如果要修改的文件已经很臃肿，在计划中包含拆分是合理的

## 反模式（绝不允许）

| 反模式 | 为什么是问题 |
|--------|------------|
| "TBD"、"TODO"、"implement later"、"fill in details" | 占位符，不是计划 |
| "Add appropriate error handling" | 没有展示怎么加 |
| "Write tests for the above"（没有实际测试代码）| 实现者不知道写什么测试 |
| "Similar to Task N" | 实现者可能乱序读任务，必须重复代码 |
| 只描述做什么不展示怎么做 | 代码步骤必须有代码块 |
| 引用未在任何任务中定义的类型/函数/方法 | 类型不一致的 bug |

## 自审清单

写完计划后，用新鲜眼光检查：

1. **规范覆盖**：规范的每个章节/需求，能指向实现它的任务吗？列出任何缺口。
2. **占位符扫描**：搜索上面的反模式，修复它们。
3. **类型一致性**：后面任务使用的类型、方法签名、属性名是否与前面定义的一致？

如果发现规范需求没有任务对应，添加任务。

## 范围检查

如果规范覆盖多个独立子系统，应该已经在 brainstorming 阶段分解了。如果没有，建议拆分为独立计划——每个计划应该产出可独立工作、可测试的软件。

## 保存位置

默认保存到 `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`。

用户偏好可覆盖此默认位置。

## 执行路径选择

计划完成后，提供两种执行方式：

**1. Subagent-Driven（推荐）** — 每个任务派一个全新子智能体，两阶段审阅，快速迭代

**2. Inline Execution** — 在当前会话内使用 `executing-plans`，批量执行+检查点审阅

详见 [05-implementation.md](05-implementation.md)。
