# 03 否定式语言

否定式语言是结构性权重中**最微妙**的类型——"绝不"/"Never" 比 "不要"/"Don't" 更强，因为否定的彻底程度不同。

## 否定强度梯度

```
  ┌──────────────────────────────────────────────────────────┐
  │  否定强度从弱到强：                                      │
  │                                                          │
  │  Level 1: 建议性否定                                     │
  │  "Try not to..." / "Avoid..."                           │
  │  → 建议不做某事，但承认可能需要例外                      │
  │                                                          │
  │  Level 2: 指示性否定                                     │
  │  "Don't..." / "Do NOT..."                               │
  │  → 指示不做某事，语气更强但仍可能有例外                  │
  │                                                          │
  │  Level 3: 绝对性否定                                     │
  │  "Never..." / "绝不"                                    │
  │  → 不承认任何例外，无条件禁止                            │
  │                                                          │
  │  Level 4: 铁律性否定                                     │
  │  "NO X WITHOUT Y" / 全大写                              │
  │  → 否定 + 条件依赖 + 格式加压 = 不可逾越               │
  │                                                          │
  │  Level 5: 道德性否定                                     │
  │  "Skip = lying" / "Bad work is worse than no work"      │
  │  → 否定被提升到道德层面——违反 = 不诚实/有害            │
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

## Superpowers 中的否定式用例

### 3.1 "Never" 清单

| 技能 | Never 用例 | 否定对象 |
|------|-----------|---------|
| subagent-driven-development | "Never start implementation on main/master" | 危险操作 |
| subagent-driven-development | "Never skip reviews" | 质量门控 |
| subagent-driven-development | "Never proceed with unfixed issues" | 带病推进 |
| subagent-driven-development | "Never dispatch multiple implementation subagents in parallel" | 并发冲突 |
| subagent-driven-development | "Never ignore an escalation" | 忽视升级 |
| subagent-driven-development | "Never make subagent read plan file" | 上下文泄漏 |
| subagent-driven-development | "Never skip scene-setting context" | 信息缺失 |
| subagent-driven-development | "Never accept 'close enough' on spec compliance" | 降标通过 |
| subagent-driven-development | "Never skip review loops" | 跳过重审 |
| subagent-driven-development | "Never let implementer self-review replace actual review" | 自审替代 |
| TDD | "Never add test-only methods to production classes" | 测试污染 |
| TDD | "Never mock without understanding dependencies" | 盲目 mock |
| code-reviewer (agent) | "Don't say 'looks good' without checking" | 虚假审阅 |
| code-reviewer (agent) | "Don't mark nitpicks as Critical" | 严重度失真 |
| code-reviewer (agent) | "Don't give feedback on code you didn't review" | 未审即评 |
| code-reviewer (agent) | "Don't be vague" | 模糊反馈 |
| receiving-code-review | "NEVER: 'You're absolutely right!'" | 表演性同意 |
| receiving-code-review | "NEVER: 'Great point!'" | 表演性赞同 |
| verification | "Never say 'should', 'probably', 'seems to'" | 模糊声称 |

### 3.2 "Do NOT" 清单

| 技能 | Do NOT 用例 | 否定对象 |
|------|-----------|---------|
| brainstorming (HARD-GATE) | "Do NOT invoke any implementation skill" | 跳过设计 |
| brainstorming (HARD-GATE) | "Do NOT write any code" | 直接实现 |
| using-superpowers | "Do NOT use the skill tool to load 'using-superpowers' again" | 冗余加载 |
| spec-reviewer | "DO NOT take their word for what they implemented" | 信任报告 |
| spec-reviewer | "DO NOT trust their claims about completeness" | 信任声称 |
| spec-reviewer | "DO NOT accept their interpretation of requirements" | 信任解读 |
| code-quality-reviewer | "Don't flag pre-existing file sizes" | 错误归因 |

### 3.3 "不可"/"不能" 暗含的否定

| 技能 | 表达 | 否定对象 |
|------|------|---------|
| writing-plans | "绝不允许" 反模式列表 | 计划中的占位符 |
| subagent-driven-development | "顺序不可颠倒" | 审阅顺序 |
| verification | "不能凭感觉声称" | 未验证声称 |
| brainstorming | "不得调用任何实现技能" | 跳过设计 |

## 否定的彻底程度对比

```
  场景：防止 AI 跳过设计直接写代码

  ┌────────────────────────────────────────────────────┐
  │  Level 1（弱否定）：                               │
  │  "Try to design before coding"                     │
  │  → AI 可能觉得"这个太简单不需要设计"              │
  │                                                    │
  │  Level 2（指示否定）：                              │
  │  "Don't write code before design"                  │
  │  → AI 可能觉得"就这一次例外"                      │
  │                                                    │
  │  Level 3（绝对否定）：                              │
  │  "Never write code before design"                  │
  │  → AI 很难找例外，但仍可能合理化                   │
  │                                                    │
  │  Level 4（铁律否定）：                              │
  │  "NO CODE WITHOUT DESIGN FIRST"                    │
  │  → 格式 + 全大写 + 否定 = 几乎不可能绕过          │
  │                                                    │
  │  Level 5（门禁否定）：                              │
  │  <HARD-GATE>                                       │
  │  Do NOT invoke any implementation skill,            │
  │  write any code, scaffold any project,             │
  │  or take any implementation action                 │
  │  until you have presented a design                 │
  │  and the user has approved it.                     │
  │  This applies to EVERY project                     │
  │  regardless of perceived simplicity.               │
  │  </HARD-GATE>                                      │
  │  → 标签 + 否定 + 全称量词 + 条件依赖 + 用户审批  │
  │  → 五重否定叠加                                    │
  └────────────────────────────────────────────────────┘
```

## 否定 vs 肯定的对比

```
  同一个意思，否定式 vs 肯定式：
  ────────────────────────────

  肯定式（弱）：
  "Always verify tests pass before claiming completion"
  → AI 知道应该做什么，但可能觉得"差不多就行"

  否定式（强）：
  "NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE"
  → AI 知道什么被禁止，违反 = 铁律违反

  差异：
  - 肯定式定义"应该做什么" → AI 有裁量空间
  - 否定式定义"不能做什么" → AI 没有裁量空间
  - 否定式的违反更明确——你要么违反了，要么没有
  - 肯定式的遵守更模糊——"我差不多做了"也是"做了"
```

## 道德性否定：最高级别

```
  ┌──────────────────────────────────────────────────────────┐
  │  道德性否定的三个用例：                                  │
  │                                                          │
  │  1. "Claiming work is complete without verification      │
  │     is dishonesty, not efficiency"                       │
  │     → 未验证声称 = 不诚实                               │
  │     → AI 极不愿意被标记为"不诚实"                       │
  │                                                          │
  │  2. "Bad work is worse than no work"                     │
  │     → 错误工作 < 没有工作                               │
  │     → 道德维度：做得差比不做更不道德                     │
  │                                                          │
  │  3. "Skip any step = lying, not verifying"               │
  │     → 跳过步骤 = 撒谎                                   │
  │     → 将流程违反直接等同于撒谎                          │
  │                                                          │
  │  为什么道德性否定最强？                                  │
  │  - AI 的训练数据中"不诚实"/"撒谎"是强负面信号          │
  │  - 道德判断比规则违反更深——违反规则可能只是"不够仔细"   │
  │  - 但"撒谎"意味着故意欺骗——AI 会被训练避免这种标记     │
  │  - 将技术行为提升到道德层面，消除了"技术上没违反"       │
  │    的灰色地带                                            │
  └──────────────────────────────────────────────────────────┘
```
