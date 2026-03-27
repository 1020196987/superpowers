---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
name_zh: 完成任务前的验证
description_zh: 当声称工作已完成、已修复或测试通过时使用，在提交或创建 PR 之前必须运行验证命令并确认输出后再做出成功声明；始终先有证据再有断言
---

# Verification Before Completion
完成前验证

## Overview
概述

Claiming work is complete without verification is dishonesty, not efficiency.

未经验证就声称工作完成是不诚实的，而非效率。

**Core principle:** Evidence before claims, always.
**核心原则：** 始终先有证据再有断言。

**Violating the letter of this rule is violating the spirit of this rule.**
**违反这条规则的文字就是违反这条规则的精神。**

## The Iron Law
铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
没有最新验证证据不得声称完成
```

If you haven't run the verification command in this message, you cannot claim it passes.

如果你没有运行此消息中的验证命令，你就不能声称它通过了。

## The Gate Function
门槛函数

```
BEFORE claiming any status or expressing satisfaction:
在声称任何状态或表达满意之前：

1. IDENTIFY: What command proves this claim?
   识别：哪个命令可以证明这个说法？
2. RUN: Execute the FULL command (fresh, complete)
   运行：执行完整命令（全新的、完整的）
3. READ: Full output, check exit code, count failures
   读取：完整输出，检查退出码，统计失败数
4. VERIFY: Does output confirm the claim?
   验证：输出是否证实了声明？
   - If NO: State actual status with evidence
     如果否：用证据说明实际状态
   - If YES: State claim WITH evidence
     如果是：用证据说明声明
5. ONLY THEN: Make the claim
   仅在那时：做出声明

Skip any step = lying, not verifying
跳过任何步骤 = 撒谎，而非验证
```

## Common Failures
常见失败

| Claim | Requires | Not Sufficient |
|------:|----------|----------------|
| Tests pass | Test command output: 0 failures | Previous run, "should pass" |
| Linter clean | Linter output: 0 errors | Partial check, extrapolation |
| Build succeeds | Build command: exit 0 | Linter passing, logs look good |
| Bug fixed | Test original symptom: passes | Code changed, assumed fixed |
| Regression test works | Red-green cycle verified | Test passes once |
| Agent completed | VCS diff shows changes | Agent reports "success" |
| Requirements met | Line-by-line checklist | Tests passing |

| 声明 | 需要 | 不充分 |
|------|------|--------|
| 测试通过 | 测试命令输出：0 个失败 | 上一次运行，"应该能通过" |
| Linter 通过 | Linter 输出：0 个错误 | 部分检查、推断 |
| 构建成功 | 构建命令：exit 0 | Linter 通过、日志看起来正常 |
| Bug 已修复 | 测试原始症状：通过 | 代码改了、假设已修复 |
| 回归测试有效 | 红绿循环已验证 | 测试通过一次 |
| Agent 完成 | VCS diff 显示变更 | Agent 报告"成功" |
| 需求已满足 | 逐行检查清单 | 测试通过 |

## Red Flags - STOP
危险信号 - 停止

- Using "should", "probably", "seems to"
  使用"应该"、"可能"、"看起来"
- Expressing satisfaction before verification ("Great!", "Perfect!", "Done!", etc.)
  在验证前表达满意（"太棒了！"、"完美！"、"完成了！"等）
- About to commit/push/PR without verification
  即将提交/推送/PR 但未验证
- Trusting agent success reports
  信任 agent 的成功报告
- Relying on partial verification
  依赖部分验证
- Thinking "just this once"
  认为"就这一次"
- Tired and wanting work over
  累了，想快点完成工作
- **ANY wording implying success without having run verification**
  **任何暗示成功但未运行验证的措辞**

## Rationalization Prevention
防止自我合理化

| Excuse | Reality |
|--------|---------|
| "Should work now" | RUN the verification |
| "I'm confident" | Confidence ≠ evidence |
| "Just this once" | No exceptions |
| "Linter passed" | Linter ≠ compiler |
| "Agent said success" | Verify independently |
| "I'm tired" | Exhaustion ≠ excuse |
| "Partial check is enough" | Partial proves nothing |
| "Different words so rule doesn't apply" | Spirit over letter |

| 借口 | 现实 |
|------|------|
| "现在应该能工作了" | 运行验证 |
| "我有信心" | 信心 ≠ 证据 |
| "就这一次" | 没有例外 |
| "Linter 通过了" | Linter ≠ 编译器 |
| "Agent 说成功了" | 独立验证 |
| "我累了" | 疲劳 ≠ 借口 |
| "部分检查就够了" | 部分证明不了任何事 |
| "措辞不同所以规则不适用" | 精神重于文字 |

## Key Patterns
关键模式

**Tests:**
**测试：**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
✅ [运行测试命令] [看到：34/34 通过] "所有测试通过"
❌ "Should pass now" / "Looks correct"
❌ "现在应该能通过了" / "看起来正确"
```

**Regression tests (TDD Red-Green):**
**回归测试（TDD 红绿循环）：**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
✅ 写 → 运行（通过）→ 撤销修复 → 运行（必须失败）→ 恢复 → 运行（通过）
❌ "I've written a regression test" (without red-green verification)
❌ "我写了回归测试"（未经红绿循环验证）
```

**Build:**
**构建：**
```
✅ [Run build] [See: exit 0] "Build passes"
✅ [运行构建] [看到：exit 0] "构建通过"
❌ "Linter passed" (linter doesn't check compilation)
❌ "Linter 通过了"（linter 不检查编译）
```

**Requirements:**
**需求：**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
✅ 重读计划 → 创建检查清单 → 逐项验证 → 报告差距或完成
❌ "Tests pass, phase complete"
❌ "测试通过，阶段完成"
```

**Agent delegation:**
**Agent 委托：**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
✅ Agent 报告成功 → 检查 VCS diff → 验证变更 → 报告实际状态
❌ Trust agent report
❌ 信任 agent 报告
```

## Why This Matters
为什么这很重要

From 24 failure memories:
来自 24 个失败记忆：

- your human partner said "I don't believe you" - trust broken
  你的人类伙伴说"我不相信你" - 信任破裂
- Undefined functions shipped - would crash
  交付了未定义的函数 - 会崩溃
- Missing requirements shipped - incomplete features
  交付了缺失的需求 - 功能不完整
- Time wasted on false completion → redirect → rework
  在虚假完成上浪费时间 → 重新定向 → 返工
- Violates: "Honesty is a core value. If you lie, you'll be replaced."
  违反："诚实是核心价值观。如果你撒谎，你将被替换。"

## When To Apply
何时适用

**ALWAYS before:**
**始终在此之前：**
- ANY variation of success/completion claims
  任何成功/完成声明的变体
- ANY expression of satisfaction
  任何满意表达
- ANY positive statement about work state
  任何关于工作状态的正向陈述
- Committing, PR creation, task completion
  提交、创建 PR、任务完成
- Moving to next task
  转向下一个任务
- Delegating to agents
  委托给 agent

**Rule applies to:**
**规则适用于：**
- Exact phrases
  精确措辞
- Paraphrases and synonyms
  改述和同义词
- Implications of success
  成功的暗示
- ANY communication suggesting completion/correctness
  任何暗示完成/正确的沟通

## The Bottom Line
底线

**No shortcuts for verification.**
**验证没有捷径。**

Run the command. Read the output. THEN claim the result.

运行命令。阅读输出。然后才声称结果。

This is non-negotiable.
这是不可协商的。
