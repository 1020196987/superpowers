---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
名称: finishing-a-development-branch
说明: 当实现完成、所有测试通过且需要决定如何整合工作时使用 - 通过提供合并、PR 或清理的结构化选项来指导开发工作的完成
---

# Finishing a Development Branch
完成开发分支

## Overview
概述

Guide completion of development work by presenting clear options and handling chosen workflow.
通过提供清晰的选项并处理所选工作流程来指导开发工作的完成。

**Core principle:** Verify tests → Present options → Execute choice → Clean up.
**核心原则：** 验证测试 → 提供选项 → 执行选择 → 清理。

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."
**开始时宣布：** "我正在使用 finishing-a-development-branch 技能来完成这项工作。"

## The Process
流程

### Step 1: Verify Tests
步骤 1：验证测试

**Before presenting options, verify tests pass:**
**在提供选项之前，请先验证测试是否通过：**

```bash
# Run project's test suite
# 运行项目测试套件
npm test / cargo test / pytest / go test ./...
```

**If tests fail:**
**如果测试失败：**
```
Tests failing (<N> failures). Must fix before completing:
测试失败（<N> 个失败）。必须先修复才能完成：

[Show failures]
[显示失败]

Cannot proceed with merge/PR until tests pass.
在测试通过之前，无法继续进行合并/PR。
```

Stop. Don't proceed to Step 2.
停止。不要继续执行步骤 2。

**If tests pass:** Continue to Step 2.
**如果测试通过：** 继续执行步骤 2。

### Step 2: Determine Base Branch
步骤 2：确定基础分支

```bash
# Try common base branches
# 尝试常见的基础分支
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Or ask: "This branch split from main - is that correct?"
或询问："此分支从 main 分叉 - 这是正确的吗？"

### Step 3: Present Options
步骤 3：提供选项

Present exactly these 4 options:
提供以下 4 个选项：

```
Implementation complete. What would you like to do?
实现完成。您想做什么？

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

1. Merge back to <base-branch> locally
   **本地合并回 <base-branch>**
2. Push and create a Pull Request
   **推送并创建 Pull Request**
3. Keep the branch as-is (I'll handle it later)
   **保持分支不变（稍后处理）**
4. Discard this work
   **放弃此工作**

Which option?
**您选择哪个选项？**

**Don't add explanation** - keep options concise.
**不要添加解释** - 保持选项简洁。

### Step 4: Execute Choice
步骤 4：执行选择

#### Option 1: Merge Locally
选项 1：本地合并

```bash
# Switch to base branch
# 切换到基础分支
git checkout <base-branch>

# Pull latest
# 拉取最新
git pull

# Merge feature branch
# 合并功能分支
git merge <feature-branch>

# Verify tests on merged result
# 验证合并后的测试
<test command>

# If tests pass
# 如果测试通过
git branch -d <feature-branch>
```

Then: Cleanup worktree (Step 5)
然后：清理工作树（步骤 5）

#### Option 2: Push and Create PR
选项 2：推送并创建 PR

```bash
# Push branch
# 推送分支
git push -u origin <feature-branch>

# Create PR
# 创建 PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
## 摘要
<2-3 bullets of what changed>
<2-3 个变更要点>

## Test Plan
## 测试计划
- [ ] <verification steps>
EOF
)"
```

Then: Cleanup worktree (Step 5)
然后：清理工作树（步骤 5）

#### Option 3: Keep As-Is
选项 3：保持不变

Report: "Keeping branch <name>. Worktree preserved at <path>."
报告："保留分支 <name>。工作树保留在 <path>。"

**Don't cleanup worktree.**
**不要清理工作树。**

#### Option 4: Discard
选项 4：放弃

**Confirm first:**
**首先确认：**
```
This will permanently delete:
这将永久删除：
- Branch <name>
- 分支 <name>
- All commits: <commit-list>
- 所有提交：<commit-list>
- Worktree at <path>
- <path> 处的工作树

Type 'discard' to confirm.
输入 'discard' 以确认。
```

Wait for exact confirmation.
等待精确确认。

If confirmed:
如果确认：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

Then: Cleanup worktree (Step 5)
然后：清理工作树（步骤 5）

### Step 5: Cleanup Worktree
步骤 5：清理工作树

**For Options 1, 2, 4:**
**适用于选项 1、2、4：**

Check if in worktree:
检查是否在 worktree 中：
```bash
git worktree list | grep $(git branch --show-current)
```

If yes:
如果存在：
```bash
git worktree remove <worktree-path>
```

**For Option 3:** Keep worktree.
**对于选项 3：** 保留工作树。

## Quick Reference
快速参考

| Option | Merge | Push | Keep Worktree | Cleanup Branch |
|--------|-------|------|---------------|----------------|
| 1. Merge locally | ✓ | - | - | ✓ |
| 2. Create PR | - | ✓ | ✓ | - |
| 3. Keep as-is | - | - | ✓ | - |
| 4. Discard | - | - | - | ✓ (force) |

| 选项 | 合并 | 推送 | 保留工作树 | 清理分支 |
|------|------|------|------------|----------|
| 1. 本地合并 | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持不变 | - | - | ✓ | - |
| 4. 放弃 | - | - | - | ✓ (强制) |

## Common Mistakes
常见错误

**Skipping test verification**
**跳过测试验证**
- **Problem:** Merge broken code, create failing PR
- **问题：** 合并损坏的代码，创建失败的 PR
- **Fix:** Always verify tests before offering options
- **修复：** 在提供选项前始终验证测试

**Open-ended questions**
**开放式问题**
- **Problem:** "What should I do next?" → ambiguous
- **问题：** "下一步该做什么？" → 模糊不清
- **Fix:** Present exactly 4 structured options
- **修复：** 提供确切的 4 个结构化选项

**Automatic worktree cleanup**
**自动清理工作树**
- **Problem:** Remove worktree when might need it (Option 2, 3)
- **问题：** 在可能需要时删除工作树（选项 2、3）
- **Fix:** Only cleanup for Options 1 and 4
- **修复：** 仅在选项 1 和 4 时清理

**No confirmation for discard**
**放弃时未确认**
- **Problem:** Accidentally delete work
- **问题：** 意外删除工作
- **Fix:** Require typed "discard" confirmation
- **修复：** 要求输入 "discard" 进行确认

## Red Flags
危险信号

**Never:**
**永远不要：**
- Proceed with failing tests
  **继续进行失败的测试**
- Merge without verifying tests on result
  **在未验证结果测试的情况下合并**
- Delete work without confirmation
  **未经确认删除工作**
- Force-push without explicit request
  **未经明确请求强制推送**

**Always:**
**始终：**
- Verify tests before offering options
  **在提供选项前验证测试**
- Present exactly 4 options
  **提供确切的 4 个选项**
- Get typed confirmation for Option 4
  **获取选项 4 的输入确认**
- Clean up worktree for Options 1 & 4 only
  **仅在选项 1 和 4 时清理工作树**

## Integration
集成

**Called by:**
**被以下调用：**
- **subagent-driven-development** (Step 7) - After all tasks complete
  **subagent-driven-development**（步骤 7）- 所有任务完成后
- **executing-plans** (Step 5) - After all batches complete
  **executing-plans**（步骤 5）- 所有批次完成后

**Pairs with:**
**配合使用：**
- **using-git-worktrees** - Cleans up worktree created by that skill
  **using-git-worktrees** - 清理由该技能创建的工作树
