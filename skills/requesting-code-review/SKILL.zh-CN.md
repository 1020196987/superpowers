---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# Requesting Code Review
请求代码审查

Dispatch superpowers:code-reviewer subagent to catch issues before they cascade. The reviewer gets precisely crafted context for evaluation — never your session's history. This keeps the reviewer focused on the work product, not your thought process, and preserves your own context for continued work.
**调度 superpowers:code-reviewer 子代理，在问题级联前捕获问题。审查者获得精确构建的上下文用于评估，而非你的会话历史。这样可以让审查者专注于工作成果，而非你的思考过程，并保留你自己的上下文以继续工作。**

**Core principle:** Review early, review often.
**核心原则：** 及早审查，持续审查。

## When to Request Review
何时请求审查

**Mandatory:**
**强制执行：**
- After each task in subagent-driven development
  **在子代理驱动开发的每个任务之后**
- After completing major feature
  **在完成主要功能之后**
- Before merge to main
  **在合并到 main 之前**

**Optional but valuable:**
**可选但有价值：**
- When stuck (fresh perspective)
  **遇到困难时（新的视角）**
- Before refactoring (baseline check)
  **重构之前（基线检查）**
- After fixing complex bug
  **修复复杂 bug 之后**

## How to Request
如何请求

**1. Get git SHAs:**
**1. 获取 git SHAs：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. Dispatch code-reviewer subagent:**
**2. 调度 code-reviewer 子代理：**

Use Task tool with superpowers:code-reviewer type, fill template at `code-reviewer.md`
**使用 superpowers:code-reviewer 类型的 Task 工具，填写 `code-reviewer.md` 中的模板**

**Placeholders:**
**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` - What you just built
  **{WHAT_WAS_IMPLEMENTED}** - 你刚刚实现的内容
- `{PLAN_OR_REQUIREMENTS}` - What it should do
  **{PLAN_OR_REQUIREMENTS}** - 它应该实现的功能
- `{BASE_SHA}` - Starting commit
  **{BASE_SHA}** - 起始提交
- `{HEAD_SHA}` - Ending commit
  **{HEAD_SHA}** - 结束提交
- `{DESCRIPTION}` - Brief summary
  **{DESCRIPTION}** - 简要说明

**3. Act on feedback:**
**3. 处理反馈：**
- Fix Critical issues immediately
  **立即修复 Critical 级别问题**
- Fix Important issues before proceeding
  **继续之前修复 Important 级别问题**
- Note Minor issues for later
  **记录 Minor 级别问题以便后续处理**
- Push back if reviewer is wrong (with reasoning)
  **如果审查者有误则反驳（需附上理由）**

## Example
示例

```
[Just completed Task 2: Add verification function]

You: Let me request code review before proceeding.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch superpowers:code-reviewer subagent]
  WHAT_WAS_IMPLEMENTED: Verification and repair functions for conversation index
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: Added verifyIndex() and repairIndex() with 4 issue types

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to proceed

You: [Fix progress indicators]
[Continue to Task 3]
```

## Integration with Workflows
与工作流集成

**Subagent-Driven Development:**
**子代理驱动开发：**
- Review after EACH task
  **每个任务后进行审查**
- Catch issues before they compound
  **在问题累积前捕获**
- Fix before moving to next task
  **修复后再进行下一任务**

**Executing Plans:**
**执行计划：**
- Review after each batch (3 tasks)
  **每批任务（3个）后进行审查**
- Get feedback, apply, continue
  **获取反馈、应用、继续**

**Ad-Hoc Development:**
**临时开发：**
- Review before merge
  **合并前审查**
- Review when stuck
  **遇到困难时审查**

## Red Flags
危险信号

**Never:**
**切勿：**
- Skip review because "it's simple"
  **因为"太简单"而跳过审查**
- Ignore Critical issues
  **忽略 Critical 级别问题**
- Proceed with unfixed Important issues
  **带着未修复的 Important 级别问题继续**
- Argue with valid technical feedback
  **与合理的技术反馈争辩**

**If reviewer wrong:**
**如果审查者有误：**
- Push back with technical reasoning
  **用技术论据反驳**
- Show code/tests that prove it works
  **展示证明其有效的代码/测试**
- Request clarification
  **请求澄清**

See template at: requesting-code-review/code-reviewer.md
请参见模板：requesting-code-review/code-reviewer.md
