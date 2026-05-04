# 阶段 ⑦：收尾（finishing-a-development-branch）

所有任务完成、测试通过后，决定如何集成工作。

## 流程

```
  ① 验证测试通过（必须！不过不往下走）
         ▼
  ② 确定基准分支 (main/master)
         ▼
  ③ 呈现 4 个选项
         ▼
  ④ 执行选择
         ▼
  ⑤ 清理 worktree（视选项而定）
```

## 选项

```
  Implementation complete. What would you like to do?

  1. Merge back to <base-branch> locally
  2. Push and create a Pull Request
  3. Keep the branch as-is (I'll handle it later)
  4. Discard this work
```

| 选项 | 合并 | 推送 | 保留 Worktree | 清理分支 |
|------|------|------|-------------|---------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持现状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓ (force) |

### 选项 1：本地合并

```bash
git checkout <base-branch>
git pull
git merge <feature-branch>
<test command>     # 验证合并后测试通过
git branch -d <feature-branch>
```

然后清理 worktree。

### 选项 2：推送并创建 PR

```bash
git push -u origin <feature-branch>
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

然后清理 worktree。

### 选项 3：保持现状

报告："Keeping branch <name>. Worktree preserved at <path>."

**不清理** worktree。

### 选项 4：丢弃

**必须先确认**：

```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

等待精确确认。确认后：

```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后清理 worktree。

## 红旗

**绝不**：
- 在测试失败时继续
- 合并前不验证测试结果
- 不确认就删除工作
- 未经明确请求强制推送

**总是**：
- 提供选项前验证测试
- 呈现精确 4 个选项
- 选项 4 需要打字确认
- 仅对选项 1 和 4 清理 worktree
