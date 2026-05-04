# 01 XML 标签包裹

XML 标签是结构性权重中**最显著的类型**——标签本身在 LLM 上下文中拥有比普通文本更高的语义权重，AI 更难将其当作"可忽略的建议"。

## 9 种 XML 标签

### 1.1 `<EXTREMELY_IMPORTANT>` — 会话启动包裹

**来源**：`hooks/session-start:35`、`.opencode/plugins/superpowers.js:73`

```html
<EXTREMELY_IMPORTANT>
You have superpowers.

**IMPORTANT: The using-superpowers skill content is included below.
It is ALREADY LOADED - you are currently following it.
Do NOT use the skill tool to load "using-superpowers" again
- that would be redundant.**

[技能完整内容]
[工具映射]
</EXTREMELY_IMPORTANT>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 标签名 | `EXTREMELY_IMPORTANT` — 名称本身就是最高级指令 |
| 位置 | 会话第一条消息——AI 看到的第一个内容 |
| 包裹范围 | 整个 bootstrap 内容——不是标注某一段，而是包裹一切 |
| 重复 | 两层：外层 `EXTREMELY_IMPORTANT` + 内层 `EXTREMELY-IMPORTANT` |
| "ALREADY LOADED" | 声明状态而非建议——"你当前正在遵循它"，不是"你应该遵循它" |

### 1.2 `<EXTREMELY-IMPORTANT>` — 1% 规则包裹

**来源**：`skills/using-superpowers/SKILL.md:10-16`

```html
<EXTREMELY-IMPORTANT>
If you think there is even a 1% chance a skill might apply
to what you are doing, you ABSOLUTELY MUST invoke the skill.

IF A SKILL APPLIES TO YOUR TASK, YOU DO NOT HAVE A CHOICE.
YOU MUST USE IT.

This is not negotiable. This is not optional.
You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 标签名 | `EXTREMELY-IMPORTANT` — 与外层标签呼应但用连字符区分 |
| 语言 | "ABSOLUTELY MUST" + "DO NOT HAVE A CHOICE" + 连续否定 |
| 逻辑 | 1% 规则——消除裁量空间，将判断阈值降到最低 |
| 位置 | 嵌套在 `EXTREMELY_IMPORTANT` 内——双重包裹 |

### 1.3 `<HARD-GATE>` — 设计审批门禁

**来源**：`skills/brainstorming/SKILL.md:12-14`

```html
<HARD-GATE>
Do NOT invoke any implementation skill, write any code,
scaffold any project, or take any implementation action
until you have presented a design and the user has approved it.
This applies to EVERY project regardless of perceived simplicity.
</HARD-GATE>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 标签名 | `HARD-GATE` — "硬门"意象，不可穿越的屏障 |
| 否定式 | "Do NOT" 开头——不是建议，是禁止 |
| 全称量词 | "EVERY project regardless of perceived simplicity"——消灭"例外" |
| 时序约束 | "until you have presented...and the user has approved"——条件未满足前绝对禁止 |

### 1.4 `<SUBAGENT-STOP>` — 子智能体隔离标记

**来源**：`skills/using-superpowers/SKILL.md:6-8`

```html
<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task,
skip this skill.
</SUBAGENT-STOP>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 标签名 | `SUBAGENT-STOP` — 命令式，"停！" |
| 身份判断 | 基于角色身份（子智能体 vs 主会话）决定行为 |
| 隔离保障 | 防止子智能体加载全套技能体系，保持聚焦 |

### 1.5 `<important-reminder>` — 旧版迁移警告

**来源**：`hooks/session-start:14`

```html
<important-reminder>
IN YOUR FIRST REPLY AFTER SEEING THIS MESSAGE YOU MUST TELL THE USER:
⚠️ **WARNING:** Superpowers now uses Claude Code's skills system.
Custom skills in ~/.config/superpowers/skills will not be read.
Move custom skills to ~/.claude/skills instead.
To make this message go away, remove ~/.config/superpowers/skills
</important-reminder>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 时序约束 | "IN YOUR FIRST REPLY"——不是"某时"，是"第一条回复" |
| 动作强制 | "YOU MUST TELL THE USER"——必须做，不是建议 |
| 消退机制 | "To make this message go away"——告诉 AI 如何让警告消失 |

### 1.6 `<agent-instructions>` — Codex 子智能体指令包裹

**来源**：`skills/using-superpowers/references/codex-tools.md:54-59`

```html
<agent-instructions>
[filled prompt content from the agent's .md file]
</agent-instructions>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 注释说明 | "Wrap instructions in XML tags — the model treats tagged blocks as authoritative" |
| 显式声明 | 直接告诉使用者：XML 标签 = 权威性 |

### 1.7 `<Good>` / `<Bad>` — 语义评价标签

**来源**：`skills/test-driven-development/SKILL.md:75-106`

```html
<Good>
```typescript
test('retries failed operations 3 times', async () => { ... });
```
Clear name, tests real behavior, one thing
</Good>

<Bad>
```typescript
test('retry works', async () => { ... });
```
Vague name, tests mock not code
</Bad>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 标签名即评价 | `Good`/`Bad` — 标签名本身就携带价值判断 |
| 结构化对比 | 不是"推荐写法"和"不推荐写法"这样的描述性标题 |
| | 而是直接用 `Good`/`Bad` 标注——评价嵌入在结构中 |

### 1.8 `<Before>` / `<After>` — 漏洞修补对比

**来源**：`skills/writing-skills/testing-skills-with-subagents.md:184-200`

```html
<Before>
Write code before test? Delete it.
</Before>

<After>
Write code before test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete
</After>
```

**权重来源**：

| 维度 | 分析 |
|------|------|
| 时间对比 | Before/After 展示演变过程——"之前的版本有漏洞" |
| 漏洞可见 | 让 AI 看到具体堵了哪些漏洞——理解"为什么加强" |
| 增量可见 | After 版本比 Before 多了什么，一目了然 |

## XML 标签的权重递进

```
  标签的权重由三个维度决定：
  ────────────────────────

  维度 1: 标签名的语义强度
  ──────────────────────
  <note>              ← 低——"注意"
  <important>         ← 中——"重要"
  <HARD-GATE>         ← 高——"硬门禁"
  <EXTREMELY_IMPORTANT> ← 最高——"极端重要"

  维度 2: 包裹范围
  ──────────────
  标注一句话          ← 低——局部约束
  标注一个段落        ← 中——区域约束
  包裹整个内容        ← 高——全局约束
  嵌套双层包裹        ← 最高——双重全局约束

  维度 3: 内容的语言强度
  ────────────────────
  包裹建议性文字      ← 弱——标签和内容不一致
  包裹命令性文字      ← 中——标签和内容一致
  包裹铁律 + 否定     ← 强——标签 + 铁律 + 否定 = 三重加压
  包裹铁律 + 否定 + 全称量词 ← 最强——四重加压
```

## 嵌套结构：双重包裹的叠加效应

```
  OpenCode 注入的实际结构：

  <EXTREMELY_IMPORTANT>              ← 外层包裹（第 1 重）
    You have superpowers.
    "ALREADY LOADED" 声明             ← 状态声明（第 2 重）

    <EXTREMELY-IMPORTANT>            ← 内层包裹（第 3 重）
      ABSOLUTELY MUST invoke          ← 绝对命令（第 4 重）
      DO NOT HAVE A CHOICE           ← 取消裁量（第 5 重）
      Not negotiable. Not optional.  ← 连续否定（第 6 重）
    </EXTREMELY-IMPORTANT>

    [技能内容]
    [工具映射]
  </EXTREMELY_IMPORTANT>

  六重加压——AI 要绕过这个，必须同时忽略六个层级的权重信号
```
