---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
name.zh-CN: 制定计划
description.zh-CN: 当你有一份多步骤任务的规格说明或需求时，在动手写代码之前使用
---

# Writing Plans
**制定计划**

## Overview
**概述**

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.
**编写全面的实施计划，假设工程师对我们的代码库没有任何背景了解，且品味有待商榷。记录他们需要了解的一切：每个任务需要修改哪些文件、代码、测试、需要查阅的文档，以及如何测试。给他们完整的计划作为小块任务。DRY（不要重复自己）。YAGNI（你不会需要它）。TDD（测试驱动开发）。频繁提交。**

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.
**假设他们是一名熟练的开发者，但对我们使用的工具集或问题领域几乎一无所知。假设他们不太擅长良好的测试设计。**

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."
**开头宣布：** "I'm using the writing-plans skill to create the implementation plan."（我正在使用制定计划技能来创建实施方案。）

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).
**上下文：** 这应该在独立的工作树中运行（由头脑风暴技能创建）。

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
**计划保存至：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)
  -（用户对计划位置的偏好会覆盖此默认值）

## Scope Check
**范围检查**

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.
**如果规格说明涵盖多个独立的子系统，那么应该在头脑风暴阶段将其拆分为子项目规格。如果之前没有拆分，建议将其拆分为独立的计划——每个子系统一个。每个计划应能独立产出可工作、可测试的软件。**

## File Structure
**文件结构**

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.
**在定义任务之前，先规划哪些文件将被创建或修改，以及每个文件负责什么。这是分解决策被确定的地方。**

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
  - 设计具有清晰边界和良好定义接口的单元。每个文件应该有一个明确的职责。
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
  - 你能最好地理解能同时放在上下文中的代码，而且当文件职责集中时，你的编辑更可靠。偏好更小、职责更集中的文件，而不是做得太多的庞大文件。
- Files that change together should live together. Split by responsibility, not by technical layer.
  - 一起变化的文件应该放在一起。按职责拆分，而非按技术层拆分。
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.
  - 在现有代码库中，遵循既定模式。如果代码库使用大文件，不要单方面重构——但如果你正在修改的文件已经变得笨重，在计划中包含拆分是合理的。

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.
**此结构指导任务分解。每个任务应产生独立的、有意义的变更。**

## Bite-Sized Task Granularity
**小块任务的粒度**

**Each step is one action (2-5 minutes):**
**每一步是一个操作（2-5 分钟）：**
- "Write the failing test" - step
  - **"Write the failing test"（编写失败的测试）** - 步骤
- "Run it to make sure it fails" - step
  - **"Run it to make sure it fails"（运行它以确保失败）** - 步骤
- "Implement the minimal code to make the test pass" - step
  - **"Implement the minimal code to make the test pass"（实现最少的代码使测试通过）** - 步骤
- "Run the tests and make sure they pass" - step
  - **"Run the tests and make sure they pass"（运行测试并确保通过）** - 步骤
- "Commit" - step
  - **"Commit"（提交）** - 步骤

## Plan Document Header
**计划文档头部**

**Every plan MUST start with this header:**
**每个计划必须以此头部开始：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure
**任务结构**

````markdown
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
````

## Remember
**请记住**

- Exact file paths always
  - **始终使用精确的文件路径**
- Complete code in plan (not "add validation")
  - **计划中包含完整代码（不要写"添加验证"）**
- Exact commands with expected output
  - **包含预期输出的精确命令**
- Reference relevant skills with @ syntax
  - **使用 @ 语法引用相关技能**
- DRY, YAGNI, TDD, frequent commits
  - **DRY（不要重复自己）、YAGNI（你不会需要它）、TDD（测试驱动开发）、频繁提交**

## Plan Review Loop
**计划审查循环**

After writing the complete plan:
**写完完整计划后：**

1. Dispatch a single plan-document-reviewer subagent (see plan-document-reviewer-prompt.md) with precisely crafted review context — never your session history. This keeps the reviewer focused on the plan, not your thought process.
   - **Dispatch a single plan-document-reviewer subagent**（派遣一个计划文档审查子代理）
   - Provide: path to the plan document, path to spec document
     - **提供：计划文档路径、规格文档路径**
2. If ❌ Issues Found: fix the issues, re-dispatch reviewer for the whole plan
   - **如果发现 ❌ 问题：修复问题，重新派遣审查者审查整个计划**
3. If ✅ Approved: proceed to execution handoff
   - **如果 ✅ 批准：继续进行执行交接**

**Review loop guidance:**
**审查循环指导：**
- Same agent that wrote the plan fixes it (preserves context)
  - **写计划的同一代理负责修复（保持上下文）**
- If loop exceeds 3 iterations, surface to human for guidance
  - **如果循环超过 3 次迭代，向人工求助**
- Reviewers are advisory — explain disagreements if you believe feedback is incorrect
  - **审查者是顾问性质——如果你认为反馈不正确，请解释分歧**

## Execution Handoff
**执行交接**

After saving the plan, offer execution choice:
**保存计划后，提供执行选择：**

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`. Two execution options:**
**"计划已完成并保存至 `docs/superpowers/plans/<filename>.md`。两种执行方式：**

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration
**1. 子代理驱动（推荐）** - 我为每个任务派遣一个新的子代理，任务之间进行审查，快速迭代

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints
**2. 内联执行** - 在此会话中使用执行计划来执行任务，带检查点的批量执行

**Which approach?"**
**选择哪种方式？**

**If Subagent-Driven chosen:**
**如果选择子代理驱动：**
- **REQUIRED SUB-SKILL:** Use superpowers:subagent-driven-development
  - **必需的子技能：** 使用 superpowers:subagent-driven-development
- Fresh subagent per task + two-stage review
  - **每个任务一个新子代理 + 两阶段审查**

**If Inline Execution chosen:**
**如果选择内联执行：**
- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans
  - **必需的子技能：** 使用 superpowers:executing-plans
- Batch execution with checkpoints for review
  - **带审查检查点的批量执行**
