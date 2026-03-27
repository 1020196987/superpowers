---
name: systematic-debugging
description: Use when encountering any bug, test failure, or unexpected behavior, before proposing fixes
name.zh-CN: 系统化调试
description.zh-CN: 遇到任何 bug、测试失败或异常行为时使用，在提出修复方案之前
---

# Systematic Debugging
# 系统化调试

## Overview
## 概述

Random fixes waste time and create new bugs. Quick patches mask underlying issues.
随机修复浪费时间并产生新的 bug。快速补丁掩盖了潜在问题。

**Core principle:** ALWAYS find root cause before attempting fixes. Symptom fixes are failure.
**核心原则：** 在尝试修复之前，始终找到根本原因。症状修复是失败。

**Violating the letter of this process is violating the spirit of debugging.**
**违反此过程的字面规定就是违反调试的精神。**

## The Iron Law
## 铁律

```
NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST
# 在首先完成根本原因调查之前，不得进行任何修复
```

If you haven't completed Phase 1, you cannot propose fixes.
如果你没有完成第一阶段，就不能提出修复方案。

## When to Use
## 何时使用

Use for ANY technical issue:
适用于任何技术问题：
- Test failures
  - **测试失败**
- Bugs in production
  - **生产环境中的 bug**
- Unexpected behavior
  - **意外行为**
- Performance problems
  - **性能问题**
- Build failures
  - **构建失败**
- Integration issues
  - **集成问题**

**Use this ESPECIALLY when:**
**特别是在以下情况下使用：**
- Under time pressure (emergencies make guessing tempting)
  - **在时间压力下（紧急情况会诱人猜测）**
- "Just one quick fix" seems obvious
  - **"只是一个快速修复"看起来很明显**
- You've already tried multiple fixes
  - **你已经尝试了多次修复**
- Previous fix didn't work
  - **之前的修复没有起作用**
- You don't fully understand the issue
  - **你没有完全理解这个问题**

**Don't skip when:**
**不要在以下情况下跳过：**
- Issue seems simple (simple bugs have root causes too)
  - **问题看起来很简单（简单的 bug 也有根本原因）**
- You're in a hurry (rushing guarantees rework)
  - **你很着急（仓促行事保证返工）**
- Manager wants it fixed NOW (systematic is faster than thrashing)
  - **经理希望现在就修复（系统化比盲目尝试更快）**

## The Four Phases
## 四个阶段

You MUST complete each phase before proceeding to the next.
你必须完成每个阶段后才能进入下一阶段。

### Phase 1: Root Cause Investigation
### 第一阶段：根本原因调查

**BEFORE attempting ANY fix:**
**在尝试任何修复之前：**

1. **Read Error Messages Carefully**
   - **仔细阅读错误信息**
   - Don't skip past errors or warnings
     - **不要跳过错误或警告**
   - They often contain the exact solution
     - **它们通常包含确切的解决方案**
   - Read stack traces completely
     - **完整阅读堆栈跟踪**
   - Note line numbers, file paths, error codes
     - **记录行号、文件路径、错误代码**

2. **Reproduce Consistently**
   - **一致地复现问题**
   - Can you trigger it reliably?
     - **你能可靠地触发它吗？**
   - What are the exact steps?
     - **确切的步骤是什么？**
   - Does it happen every time?
     - **每次都会发生吗？**
   - If not reproducible → gather more data, don't guess
     - **如果无法复现 → 收集更多数据，不要猜测**

3. **Check Recent Changes**
   - **检查最近的变更**
   - What changed that could cause this?
     - **什么变更可能导致这个问题？**
   - Git diff, recent commits
     - **Git diff，最近的提交**
   - New dependencies, config changes
     - **新依赖项，配置变更**
   - Environmental differences
     - **环境差异**

4. **Gather Evidence in Multi-Component Systems**
   - **在多组件系统中收集证据**

   **WHEN system has multiple components (CI → build → signing, API → service → database):**
   **当系统有多个组件时（CI → 构建 → 签名，API → 服务 → 数据库）：**

   **BEFORE proposing fixes, add diagnostic instrumentation:**
   **在提出修复方案之前，添加诊断工具：**
   ```
   # 对于每个组件边界：
   For EACH component boundary:
     - Log what data enters component
     - Log what data exits component
     - Verify environment/config propagation
     - Check state at each layer

   Run once to gather evidence showing WHERE it breaks
   THEN analyze evidence to identify failing component
   THEN investigate that specific component
   # 运行一次收集证据，显示在哪里出问题
   # 然后分析证据以识别故障组件
   # 然后调查该特定组件
   ```

   **Example (multi-layer system):**
   **示例（多层系统）：**
   ```bash
   # Layer 1: Workflow
   # 第 1 层：工作流
   echo "=== Secrets available in workflow: ==="
   echo "IDENTITY: ${IDENTITY:+SET}${IDENTITY:-UNSET}"

   # Layer 2: Build script
   # 第 2 层：构建脚本
   echo "=== Env vars in build script: ==="
   env | grep IDENTITY || echo "IDENTITY not in environment"

   # Layer 3: Signing script
   # 第 3 层：签名脚本
   echo "=== Keychain state: ==="
   security list-keychains
   security find-identity -v

   # Layer 4: Actual signing
   # 第 4 层：实际签名
   codesign --sign "$IDENTITY" --verbose=4 "$APP"
   ```

   **This reveals:** Which layer fails (secrets → workflow ✓, workflow → build ✗)
   **这揭示了：** 哪一层失败（secrets → workflow ✓, workflow → build ✗）

5. **Trace Data Flow**
   - **追踪数据流**

   **WHEN error is deep in call stack:**
   **当错误在调用栈深处时：**

   See `root-cause-tracing.md` in this directory for the complete backward tracing technique.
   有关完整的向后追踪技术，请参阅此目录中的 `root-cause-tracing.md`。

   **Quick version:**
   **快速版本：**
   - Where does bad value originate?
     - **错误值从哪里产生？**
   - What called this with bad value?
     - **是什么用错误值调用了它？**
   - Keep tracing up until you find the source
     - **一直追踪直到找到源头**
   - Fix at source, not at symptom
     - **在源头修复，而不是在症状处修复**

### Phase 2: Pattern Analysis
### 第二阶段：模式分析

**Find the pattern before fixing:**
**在修复之前找到模式：**

1. **Find Working Examples**
   - **找到有效的示例**
   - Locate similar working code in same codebase
     - **在同一代码库中找到相似的有效代码**
   - What works that's similar to what's broken?
     - **什么与损坏的类似但有效？**

2. **Compare Against References**
   - **与参考实现对比**
   - If implementing pattern, read reference implementation COMPLETELY
     - **如果实现模式，请完整阅读参考实现**
   - Don't skim - read every line
     - **不要略读 —— 逐行阅读**
   - Understand the pattern fully before applying
     - **在应用之前完全理解该模式**

3. **Identify Differences**
   - **识别差异**
   - What's different between working and broken?
     - **有效的和损坏的之间有什么不同？**
   - List every difference, however small
     - **列出每个差异，无论多小**
   - Don't assume "that can't matter"
     - **不要假设"那不重要"**

4. **Understand Dependencies**
   - **理解依赖关系**
   - What other components does this need?
     - **这还需要什么其他组件？**
   - What settings, config, environment?
     - **什么设置、配置、环境？**
   - What assumptions does it make?
     - **它做了什么假设？**

### Phase 3: Hypothesis and Testing
### 第三阶段：假设与测试

**Scientific method:**
**科学方法：**

1. **Form Single Hypothesis**
   - **形成单一假设**
   - State clearly: "I think X is the root cause because Y"
     - **清楚地陈述："我认为 X 是根本原因，因为 Y"**
   - Write it down
     - **把它写下来**
   - Be specific, not vague
     - **要具体，不要模糊**

2. **Test Minimally**
   - **最小化测试**
   - Make the SMALLEST possible change to test hypothesis
     - **做最小的可能更改来测试假设**
   - One variable at a time
     - **一次只改一个变量**
   - Don't fix multiple things at once
     - **不要同时修复多件事**

3. **Verify Before Continuing**
   - **在继续之前验证**
   - Did it work? Yes → Phase 4
     - **起作用了吗？是 → 第四阶段**
   - Didn't work? Form NEW hypothesis
     - **没起作用？形成新的假设**
   - DON'T add more fixes on top
     - **不要在原有基础上添加更多修复**

4. **When You Don't Know**
   - **当你不知道的时候**
   - Say "I don't understand X"
     - **说"我不理解 X"**
   - Don't pretend to know
     - **不要假装知道**
   - Ask for help
     - **寻求帮助**
   - Research more
     - **做更多研究**

### Phase 4: Implementation
### 第四阶段：实施

**Fix the root cause, not the symptom:**
**修复根本原因，而不是症状：**

1. **Create Failing Test Case**
   - **创建失败的测试用例**
   - Simplest possible reproduction
     - **最简单的复现方式**
   - Automated test if possible
     - **如果可能，使用自动化测试**
   - One-off test script if no framework
     - **如果没有框架，使用一次性测试脚本**
   - MUST have before fixing
     - **修复前必须有**
   - Use the `superpowers:test-driven-development` skill for writing proper failing tests
     - **使用 `superpowers:test-driven-development` 技能来编写正确的失败测试**

2. **Implement Single Fix**
   - **实施单一修复**
   - Address the root cause identified
     - **解决已识别的根本原因**
   - ONE change at a time
     - **一次只做一个更改**
   - No "while I'm here" improvements
     - **不要"顺手"改进**
   - No bundled refactoring
     - **不要捆绑重构**

3. **Verify Fix**
   - **验证修复**
   - Test passes now?
     - **现在测试通过了吗？**
   - No other tests broken?
     - **没有其他测试被破坏吗？**
   - Issue actually resolved?
     - **问题真的解决了吗？**

4. **If Fix Doesn't Work**
   - **如果修复不起作用**
   - STOP
     - **停止**
   - Count: How many fixes have you tried?
     - **计数：你尝试了多少次修复？**
   - If < 3: Return to Phase 1, re-analyze with new information
     - **如果 < 3：返回第一阶段，用新信息重新分析**
   - **If ≥ 3: STOP and question the architecture (step 5 below)**
     - **如果 ≥ 3：停止并质疑架构（见下面的第 5 步）**
   - **DON'T attempt Fix #4 without architectural discussion**
     - **未经架构讨论不要尝试第 4 次修复**

5. **If 3+ Fixes Failed: Question Architecture**
   - **如果 3 次以上修复失败：质疑架构**

   **Pattern indicating architectural problem:**
   **表明架构问题的模式：**
   - Each fix reveals new shared state/coupling/problem in different place
     - **每次修复都在不同地方揭示新的共享状态/耦合/问题**
   - Fixes require "massive refactoring" to implement
     - **修复需要"大规模重构"才能实施**
   - Each fix creates new symptoms elsewhere
     - **每次修复在其他地方产生新症状**

   **STOP and question fundamentals:**
   **停止并质疑基本原理：**
   - Is this pattern fundamentally sound?
     - **这个模式从根本上来说是合理的吗？**
   - Are we "sticking with it through sheer inertia"?
     - **我们是在"纯粹因为惯性而坚持"吗？**
   - Should we refactor architecture vs. continue fixing symptoms?
     - **我们应该重构架构还是继续修复症状？**

   **Discuss with your human partner before attempting more fixes**
   **在尝试更多修复之前，与你的合作伙伴讨论**

   This is NOT a failed hypothesis - this is a wrong architecture.
   这不是假设失败 —— 这是架构错误。

## Red Flags - STOP and Follow Process
## 危险信号 —— 停止并遵循流程

If you catch yourself thinking:
如果你发现自己有如下想法：
- "Quick fix for now, investigate later"
  - **"现在先快速修复，之后再调查"**
- "Just try changing X and see if it works"
  - **"就试着改一下 X，看看有没有用"**
- "Add multiple changes, run tests"
  - **"添加多个更改，然后运行测试"**
- "Skip the test, I'll manually verify"
  - **"跳过测试，我自己手动验证"**
- "It's probably X, let me fix that"
  - **"大概是 X，我来解决它"**
- "I don't fully understand but this might work"
  - **"我没有完全理解，但这可能有用"**
- "Pattern says X but I'll adapt it differently"
  - **"模式说 X，但我会做不同的调整"**
- "Here are the main problems: [lists fixes without investigation]"
  - **"主要问题是：[列出修复方案但没有调查]"**
- Proposing solutions before tracing data flow
  - **在追踪数据流之前提出解决方案**
- **"One more fix attempt" (when already tried 2+)**
  - **"再试一次修复"（已经尝试了 2 次以上）**
- **Each fix reveals new problem in different place**
  - **每次修复都在不同地方揭示新问题**

**ALL of these mean: STOP. Return to Phase 1.**
**以上所有都意味着：停止。返回第一阶段。**

**If 3+ fixes failed:** Question the architecture (see Phase 4.5)
**如果 3 次以上修复失败：** 质疑架构（见第四阶段第 5 步）

## your human partner's Signals You're Doing It Wrong
## 你的合作伙伴暗示你做错了

**Watch for these redirections:**
**注意这些重定向：**
- "Is that not happening?" - You assumed without verifying
  - **"那没有发生吗？" —— 你没有验证就假设了**
- "Will it show us...?" - You should have added evidence gathering
  - **"它会显示给我们...?" —— 你应该添加证据收集**
- "Stop guessing" - You're proposing fixes without understanding
  - **"停止猜测" —— 你在没有理解的情况下提出修复方案**
- "Ultrathink this" - Question fundamentals, not just symptoms
  - **"深度思考这个" —— 质疑基本原理，而不仅仅是症状**
- "We're stuck?" (frustrated) - Your approach isn't working
  - **"我们卡住了？"（沮丧地）—— 你的方法不奏效**

**When you see these:** STOP. Return to Phase 1.
**当你看到这些时：** 停止。返回第一阶段。

## Common Rationalizations
## 常见的自我合理化

| Excuse | Reality |
| --- | --- |
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| **借口** | **现实** |
| "Issue is simple, don't need process" | Simple issues have root causes too. Process is fast for simple bugs. |
| **"问题很简单，不需要流程"** | **简单的问题也有根本原因。流程对简单 bug 很快。** |
| "Emergency, no time for process" | Systematic debugging is FASTER than guess-and-check thrashing. |
| **"紧急情况，没时间走流程"** | **系统化调试比猜测-检查的盲目尝试更快。** |
| "Just try this first, then investigate" | First fix sets the pattern. Do it right from the start. |
| **"先试试这个，然后再调查"** | **第一次修复决定了模式。从一开始就做对。** |
| "I'll write test after confirming fix works" | Untested fixes don't stick. Test first proves it. |
| **"我会在确认修复有效后再写测试"** | **未经测试的修复不会持久。先测试才能证明。** |
| "Multiple fixes at once saves time" | Can't isolate what worked. Causes new bugs. |
| **"一次做多个修复节省时间"** | **无法隔离哪个起作用了。会导致新的 bug。** |
| "Reference too long, I'll adapt the pattern" | Partial understanding guarantees bugs. Read it completely. |
| **"参考太长了，我会调整模式"** | **部分理解保证会产生 bug。完整阅读。** |
| "I see the problem, let me fix it" | Seeing symptoms ≠ understanding root cause. |
| **"我看到问题了，让我来修复"** | **看到症状 ≠ 理解根本原因。** |
| "One more fix attempt" (after 2+ failures) | 3+ failures = architectural problem. Question pattern, don't fix again. |
| **"再试一次修复"（2 次以上失败后）** | **3 次以上失败 = 架构问题。质疑模式，不要再修复。** |

## Quick Reference
## 快速参考

| Phase | Key Activities | Success Criteria |
| --- | --- | --- |
| **1. Root Cause** | Read errors, reproduce, check changes, gather evidence | Understand WHAT and WHY |
| **1. 根本原因** | **阅读错误、复现、检查变更、收集证据** | **理解是什么和为什么** |
| **2. Pattern** | Find working examples, compare | Identify differences |
| **2. 模式** | **找到有效示例、对比** | **识别差异** |
| **3. Hypothesis** | Form theory, test minimally | Confirmed or new hypothesis |
| **3. 假设** | **形成理论、最小化测试** | **已确认或新假设** |
| **4. Implementation** | Create test, fix, verify | Bug resolved, tests pass |
| **4. 实施** | **创建测试、修复、验证** | **Bug 已解决，测试通过** |

## When Process Reveals "No Root Cause"
## 当流程揭示"没有根本原因"时

If systematic investigation reveals issue is truly environmental, timing-dependent, or external:
如果系统化调查表明问题确实是环境相关的、时间相关的或外部的：

1. You've completed the process
   - **你已经完成了流程**
2. Document what you investigated
   - **记录你调查了什么**
3. Implement appropriate handling (retry, timeout, error message)
   - **实施适当的处理（重试、超时、错误消息）**
4. Add monitoring/logging for future investigation
   - **添加监控/日志以便将来调查**

**But:** 95% of "no root cause" cases are incomplete investigation.
**但是：** 95% 的"没有根本原因"案例是调查不完整。

## Supporting Techniques
## 支持技术

These techniques are part of systematic debugging and available in this directory:
这些技术是系统化调试的一部分，可在此目录中获取：

- **`root-cause-tracing.md`** - Trace bugs backward through call stack to find original trigger
  - **`root-cause-tracing.md`** - 通过调用栈向后追踪 bug 以找到原始触发点
- **`defense-in-depth.md`** - Add validation at multiple layers after finding root cause
  - **`defense-in-depth.md`** - 找到根本原因后在多个层级添加验证
- **`condition-based-waiting.md`** - Replace arbitrary timeouts with condition polling
  - **`condition-based-waiting.md`** - 用条件轮询替换任意超时

**Related skills:**
**相关技能：**
- **superpowers:test-driven-development** - For creating failing test case (Phase 4, Step 1)
  - **superpowers:test-driven-development** - 用于创建失败测试用例（第四阶段第 1 步）
- **superpowers:verification-before-completion** - Verify fix worked before claiming success
  - **superpowers:verification-before-completion** - 在宣称成功之前验证修复是否有效

## Real-World Impact
## 实际影响

From debugging sessions:
来自调试会话的数据：
- Systematic approach: 15-30 minutes to fix
  - **系统化方法：15-30 分钟修复**
- Random fixes approach: 2-3 hours of thrashing
  - **随机修复方法：2-3 小时的盲目尝试**
- First-time fix rate: 95% vs 40%
  - **首次修复率：95% vs 40%**
- New bugs introduced: Near zero vs common
  - **引入新 bug：接近零 vs 常见**
