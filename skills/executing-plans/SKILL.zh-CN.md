---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# Executing Plans
# 执行计划

## Overview
## 概述

Load plan, review critically, execute all tasks, report when complete.
加载计划，仔细审查，执行所有任务，完成后报告。

**Announce at start:** "I'm using the executing-plans skill to implement this plan."
**开头声明：** "我正在使用执行计划技能来实现此计划。"

**Note:** Tell your human partner that Superpowers works much better with access to subagents. The quality of its work will be significantly higher if run on a platform with subagent support (such as Claude Code or Codex). If subagents are available, use superpowers:subagent-driven-development instead of this skill.
**注意：** 告知你的合作伙伴，Superpowers 在有子代理支持的情况下运行效果会更好。如果在支持子代理的平台上运行（如 Claude Code 或 Codex），工作质量会显著提高。如果有子代理可用，请使用 superpowers:subagent-driven-development 而不是此技能。

## The Process
## 执行流程

### Step 1: Load and Review Plan
### 步骤 1: 加载并审查计划

1. Read plan file
  读取计划文件
2. Review critically - identify any questions or concerns about the plan
  仔细审查 - 识别计划中的任何问题或疑虑
3. If concerns: Raise them with your human partner before starting
  如有疑虑：在开始前向合作伙伴提出
4. If no concerns: Create TodoWrite and proceed
  如无疑虑：创建 TodoWrite 并继续

### Step 2: Execute Tasks
### 步骤 2: 执行任务

For each task:
对于每个任务：

1. Mark as in_progress
  标记为进行中
2. Follow each step exactly (plan has bite-sized steps)
  严格遵循每个步骤（计划中包含小粒度步骤）
3. Run verifications as specified
  按规定运行验证
4. Mark as completed
  标记为已完成

### Step 3: Complete Development
### 步骤 3: 完成开发

After all tasks complete and verified:
所有任务完成并验证后：

- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
  声明："我正在使用完成开发分支技能来完成此工作。"
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
  **必需的子技能：** 使用 superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice
  按照该技能验证测试、展示选项并执行选择

## When to Stop and Ask for Help
## 何时停止并寻求帮助

**STOP executing immediately when:**
**立即停止执行：**

- Hit a blocker (missing dependency, test fails, instruction unclear)
  遇到阻碍（依赖缺失、测试失败、指令不明确）
- Plan has critical gaps preventing starting
  计划存在关键缺陷，无法开始
- You don't understand an instruction
  不理解某条指令
- Verification fails repeatedly
  验证反复失败

**Ask for clarification rather than guessing.**
**有疑问时请求澄清，而不是猜测。**

## When to Revisit Earlier Steps
## 何时回溯早期步骤

**Return to Review (Step 1) when:**
**返回审查（步骤 1）：**

- Partner updates the plan based on your feedback
  合作伙伴根据你的反馈更新了计划
- Fundamental approach needs rethinking
  基本方法需要重新思考

**Don't force through blockers** - stop and ask.
**不要强行突破阻碍** - 停下来并寻求帮助。

## Remember
## 注意事项

- Review plan critically first
  首先仔细审查计划
- Follow plan steps exactly
  严格遵循计划步骤
- Don't skip verifications
  不要跳过验证
- Reference skills when plan says to
  计划中提到时引用相关技能
- Stop when blocked, don't guess
  遇到阻碍时停止，不要猜测
- Never start implementation on main/master branch without explicit user consent
  未经用户明确同意，绝不能在 main/master 分支上开始实现

## Integration
## 集成

**Required workflow skills:**
**必需的工作流技能：**

- **superpowers:using-git-worktrees** - REQUIRED: Set up isolated workspace before starting
  **superpowers:using-git-worktrees** - 必需：在开始前设置隔离的工作空间
- **superpowers:writing-plans** - Creates the plan this skill executes
  **superpowers:writing-plans** - 创建此技能执行的计划
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
  **superpowers:finishing-a-development-branch** - 在所有任务完成后完成开发
