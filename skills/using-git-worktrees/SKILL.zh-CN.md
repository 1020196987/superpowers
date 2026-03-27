---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
name_zh: using-git-worktrees
description_zh: 在开始需要与当前工作区隔离的功能工作或执行实现计划之前使用——创建具有智能目录选择和安全验证的隔离 git worktree
---

# Using Git Worktrees
# 使用 Git Worktrees

## Overview
## 概述

Git worktrees create isolated workspaces sharing the same repository, allowing work on multiple branches simultaneously without switching.
Git worktrees 创建共享同一仓库的隔离工作区，允许同时在多个分支上工作而无需切换。

**Core principle:** Systematic directory selection + safety verification = reliable isolation.
**核心原则：** 系统化目录选择 + 安全验证 = 可靠的隔离。

**Announce at start:** "I'm using the using-git-worktrees skill to set up an isolated workspace."
**开始时宣布：** "I'm using the using-git-worktrees skill to set up an isolated workspace."

## Directory Selection Process
## 目录选择流程

Follow this priority order:
按以下优先级顺序：

### 1. Check Existing Directories
### 1. 检查现有目录

```bash
# Check in priority order
# 按优先级顺序检查
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
# 首选（隐藏）
ls -d worktrees 2>/dev/null      # Alternative
# 备选
```

**If found:** Use that directory. If both exist, `.worktrees` wins.
**如果找到：** 使用该目录。如果两者都存在，`.worktrees` 优先。

### 2. Check CLAUDE.md
### 2. 检查 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**If preference specified:** Use it without asking.
**如果指定了偏好：** 直接使用，无需询问。

### 3. Ask User
### 3. 询问用户

If no directory exists and no CLAUDE.md preference:
如果没有目录存在且没有 CLAUDE.md 偏好：

```
No worktree directory found. Where should I create worktrees?
未找到 worktree 目录。应该在哪里创建 worktree？

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)
  2. ~/.config/superpowers/worktrees/<project-name>/ (全局位置)

Which would you prefer?
您选择哪一个？
```

## Safety Verification
## 安全验证

### For Project-Local Directories (.worktrees or worktrees)
### 项目本地目录（.worktrees 或 worktrees）

**MUST verify directory is ignored before creating worktree:**
**创建 worktree 前必须验证目录已被忽略：**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
# 检查目录是否被忽略（遵循 local、global 和 system gitignore）
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**If NOT ignored:**
**如果没有被忽略：**

Per Jesse's rule "Fix broken things immediately":
根据 Jesse 的规则 "立即修复损坏的东西"：

1. Add appropriate line to .gitignore
1. 添加适当的行到 .gitignore
2. Commit the change
2. 提交更改
3. Proceed with worktree creation
3. 继续创建 worktree

**Why critical:** Prevents accidentally committing worktree contents to repository.
**为什么关键：** 防止意外将 worktree 内容提交到仓库。

### For Global Directory (~/.config/superpowers/worktrees)
### 全局目录（~/.config/superpowers/worktrees）

No .gitignore verification needed - outside project entirely.
无需 .gitignore 验证——完全在项目外部。

## Creation Steps
## 创建步骤

### 1. Detect Project Name
### 1. 检测项目名称

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. Create Worktree
### 2. 创建 Worktree

```bash
# Determine full path
# 确定完整路径
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
# 使用新分支创建 worktree
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. Run Project Setup
### 3. 运行项目设置

Auto-detect and run appropriate setup:
自动检测并运行适当的设置：

```bash
# Node.js
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. Verify Clean Baseline
### 4. 验证干净的基线

Run tests to ensure worktree starts clean:
运行测试以确保 worktree 从干净状态开始：

```bash
# Examples - use project-appropriate command
# 示例——使用项目适当的命令
npm test
cargo test
pytest
go test ./...
```

**If tests fail:** Report failures, ask whether to proceed or investigate.
**如果测试失败：** 报告失败，询问是继续还是调查。

**If tests pass:** Report ready.
**如果测试通过：** 报告就绪。

### 5. Report Location
### 5. 报告位置

```
Worktree ready at <full-path>
Worktree 已就绪，位于 <完整路径>
Tests passing (<N> tests, 0 failures)
测试通过（<N> 个测试，0 失败）
Ready to implement <feature-name>
准备好实现 <功能名称>
```

## Quick Reference
## 快速参考

| Situation | Action |
| 情况 | 操作 |
|-----------|--------|
| `.worktrees/` exists | Use it (verify ignored) |
| `.worktrees/` 存在 | 使用它（验证已忽略） |
| `worktrees/` exists | Use it (verify ignored) |
| `worktrees/` 存在 | 使用它（验证已忽略） |
| Both exist | Use `.worktrees/` |
| 两者都存在 | 使用 `.worktrees/` |
| Neither exists | Check CLAUDE.md → Ask user |
| 两者都不存在 | 检查 CLAUDE.md → 询问用户 |
| Directory not ignored | Add to .gitignore + commit |
| 目录未被忽略 | 添加到 .gitignore + 提交 |
| Tests fail during baseline | Report failures + ask |
| 基线测试失败 | 报告失败 + 询问 |
| No package.json/Cargo.toml | Skip dependency install |
| 没有 package.json/Cargo.toml | 跳过依赖安装 |

## Common Mistakes
## 常见错误

### Skipping ignore verification
### 跳过忽略验证

- **Problem:** Worktree contents get tracked, pollute git status
- **问题：** Worktree 内容被跟踪，污染 git 状态
- **Fix:** Always use `git check-ignore` before creating project-local worktree
- **修复：** 在创建项目本地 worktree 前始终使用 `git check-ignore`

### Assuming directory location
### 假设目录位置

- **Problem:** Creates inconsistency, violates project conventions
- **问题：** 造成不一致，违反项目约定
- **Fix:** Follow priority: existing > CLAUDE.md > ask
- **修复：** 遵循优先级：现有 > CLAUDE.md > 询问

### Proceeding with failing tests
### 在测试失败时继续

- **Problem:** Can't distinguish new bugs from pre-existing issues
- **问题：** 无法区分新 bug 和已有问题
- **Fix:** Report failures, get explicit permission to proceed
- **修复：** 报告失败，获得明确许可后再继续

### Hardcoding setup commands
### 硬编码设置命令

- **Problem:** Breaks on projects using different tools
- **问题：** 在使用不同工具的项目上会失败
- **Fix:** Auto-detect from project files (package.json, etc.)
- **修复：** 从项目文件自动检测（package.json 等）

## Example Workflow
## 示例工作流

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.
您：I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[检查 .worktrees/ - 存在]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[验证已忽略 - git check-ignore 确认 .worktrees/ 已被忽略]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[创建 worktree：git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[运行 npm install]
[Run npm test - 47 passing]
[运行 npm test - 47 通过]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Worktree 已就绪，位于 /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
测试通过（47 个测试，0 失败）
Ready to implement auth feature
准备好实现 auth 功能
```

## Red Flags
## 危险信号

**Never:**
**永远不要：**

- Create worktree without verifying it's ignored (project-local)
- 不验证忽略状态就创建 worktree（项目本地）
- Skip baseline test verification
- 跳过基线测试验证
- Proceed with failing tests without asking
- 不询问就继续处理失败的测试
- Assume directory location when ambiguous
- 在不明确时假设目录位置
- Skip CLAUDE.md check
- 跳过 CLAUDE.md 检查

**Always:**
**始终：**

- Follow directory priority: existing > CLAUDE.md > ask
- 遵循目录优先级：现有 > CLAUDE.md > 询问
- Verify directory is ignored for project-local
- 验证项目本地目录已被忽略
- Auto-detect and run project setup
- 自动检测并运行项目设置
- Verify clean test baseline
- 验证干净的测试基线

## Integration
## 集成

**Called by:**
**由以下调用：**

- **brainstorming** (Phase 4) - REQUIRED when design is approved and implementation follows
- **brainstorming**（第 4 阶段）——设计批准且进入实现阶段时必须使用
- **subagent-driven-development** - REQUIRED before executing any tasks
- **subagent-driven-development**——执行任何任务前必须使用
- **executing-plans** - REQUIRED before executing any tasks
- **executing-plans**——执行任何任务前必须使用
- Any skill needing isolated workspace
- 需要隔离工作区的任何技能

**Pairs with:**
**配合使用：**

- **finishing-a-development-branch** - REQUIRED for cleanup after work complete
- **finishing-a-development-branch**——工作完成后清理时必须使用
