# Superpowers 项目详细总结

## 项目概述

**Superpowers** 是一个面向 AI 编码代理的完整软件开发工作流程框架，基于一组可组合的"技能"(Skills) 和初始指令构建，托管在 GitHub 上（`obra/superpowers`），当前版本 **v5.0.5**。

核心设计理念：AI 代理在开始写代码之前，先通过结构化的技能流程（头脑风暴 -> 规划 -> 执行 -> 审查）确保设计清晰、代码高质量。

**核心哲学**：
- 测试优先（TDD）
- 系统化优于即兴
- 复杂度降低是主要目标
- 证据优于声称

---

## 1. 项目整体结构

```
superpowers/
├── skills/          # 核心技能库（15个技能目录）
├── agents/          # 代理定义
├── commands/        # 命令（已废弃）
├── hooks/           # 会话钩子
├── docs/            # 文档
├── openspec/        # 项目设计规范（OpenSpec 格式）
├── tests/           # 测试套件
├── __learn__/       # 学习总结（本文件）
├── .claude-plugin/ # Claude Code 插件配置
├── .cursor-plugin/  # Cursor 插件配置
├── .codex/          # Codex 安装脚本
├── .opencode/       # OpenCode 插件
├── package.json     # npm 包定义
├── README.md        # 项目主页
├── CHANGELOG.md     # 变更日志
├── RELEASE-NOTES.md # 发布说明
├── GEMINI.md        # Gemini CLI 配置
└── gemini-extension.json  # Gemini 扩展配置
```

---

## 2. 核心配置文件

### `.claude-plugin/plugin.json` — Claude Code 官方插件配置

```json
{
  "name": "superpowers",
  "version": "5.0.2",
  "skills": "./skills/",
  "agents": "./agents/",
  "commands": "./commands/",
  "hooks": "./hooks/hooks-cursor.json"
}
```

### `package.json` — npm 包定义

```json
{
  "name": "superpowers",
  "version": "5.0.4",
  "main": ".opencode/plugins/superpowers.js"
}
```

### `GEMINI.md` — Gemini CLI 引导文件

Gemini CLI 通过此文件加载技能系统的引导内容，引用 `skills/using-superpowers/SKILL.md` 和 `gemini-tools.md`。

---

## 3. skills/ 目录 — 技能库（核心）

共 **15 个技能**，分为五类：测试、调试、协作、流程和元技能。

### 3.1 测试类技能

#### `skills/test-driven-development/` — 测试驱动开发（TDD）

**铁律**：`没有失败的测试，就不能写生产代码`。

三阶段流程：
1. **RED阶段**：写一个最小化的失败测试
2. **GREEN阶段**：写最少的代码让测试通过
3. **REFACTOR阶段**：清理代码，保持测试绿色

包含：
- `SKILL.md` — TDD 技能主文件
- `testing-anti-patterns.md` — 测试反模式参考（5 种常见错误）

#### `skills/verification-before-completion/` — 完成前验证

**铁律**：`没有新鲜验证证据就不能声称完成`。

闸门功能：
1. 识别：什么命令能证明这个声明？
2. 运行：执行完整命令
3. 读取：完整输出，检查退出码，统计失败数
4. 验证：输出是否确认了声明？

---

### 3.2 调试类技能

#### `skills/systematic-debugging/` — 系统性调试

**铁律**：`不找到根本原因，就不能提出修复方案`。

四阶段调试过程：
1. **Phase 1 - 根本原因调查**：仔细阅读错误信息、一致性重现、检查最近变更
2. **Phase 2 - 模式分析**：找类似工作的代码、比较差异、理解依赖
3. **Phase 3 - 假设与测试**：形成单一假设，最小化修改测试、验证后继续
4. **Phase 4 - 实现**：创建失败测试用例、单次修复一个根因、验证

包含：
- `SKILL.md` — 调试技能主文件
- `root-cause-tracing.md` — 根本原因追踪技术
- `defense-in-depth.md` — 多层防御验证技术
- `condition-based-waiting.md` — 条件等待技术
- `find-polluter.sh` — 测试污染定位脚本
- `condition-based-waiting-example.ts` — 条件等待助手函数示例

---

### 3.3 协作类技能

#### `skills/brainstorming/` — 头脑风暴

**铁律（HARD-GATE）**：在呈现设计方案并获得用户批准之前，不得调用任何实现技能、编写任何代码、搭建任何项目。

流程：
1. 探索项目上下文
2. 如果涉及视觉问题，提供 Visual Companion
3. 一次一个地提问澄清问题
4. 提出 2-3 种方案及权衡
5. 分段呈现设计方案
6. 将设计文档写入 `docs/superpowers/specs/`
7. **Spec 审查循环**：派发 spec-document-reviewer 子代理，最多 3 轮
8. 用户审查书面 spec
9. 转向 writing-plans 技能

包含：
- `SKILL.md` — 头脑风暴技能主文件
- `visual-companion.md` — 可视化伴侣指南（基于浏览器的原型展示工具）
- `spec-document-reviewer-prompt.md` — Spec 审查子代理提示模板

#### `skills/writing-plans/` — 编写实现计划

核心原则：假设执行者对代码库零背景知识，给出完整计划——每个任务的精确文件路径、完整代码、验证步骤、测试命令。

任务粒度：每个步骤 2-5 分钟，包含 TDD 循环。

包含：
- `SKILL.md` — 计划编写技能主文件
- `plan-document-reviewer-prompt.md` — 计划审查子代理提示模板

#### `skills/executing-plans/` — 执行计划

用于在独立会话中执行书面实现计划（带审查检查点）。

注：如果平台支持 subagent，推荐使用 `subagent-driven-development` 替代。

#### `skills/subagent-driven-development/` — 子代理驱动开发（推荐）

通过为每个任务派发新的 subagent 实现计划，每个任务后进行两阶段审查。

两阶段审查：
1. **Spec 合规性审查**：验证是否构建了请求的内容
2. **代码质量审查**：验证实现是否构建良好

包含：
- `SKILL.md` — 子代理驱动开发技能主文件
- `implementer-prompt.md` — 实施者子代理提示模板
- `spec-reviewer-prompt.md` — Spec 合规性审查提示模板
- `code-quality-reviewer-prompt.md` — 代码质量审查提示模板

#### `skills/dispatching-parallel-agents/` — 并行代理派发

当有 2+ 个独立任务可以无共享状态并行处理时使用。

核心原则：每个独立问题域派发一个代理，让他们并发工作。

#### `skills/requesting-code-review/` — 请求代码审查

在完成任务、实现重大功能、或合并前验证工作是否符合要求时使用。

派发 `code-reviewer` 子代理，检查代码质量、架构、测试。

#### `skills/receiving-code-review/` — 接收代码审查

在收到代码审查反馈后、实现建议之前使用。

核心原则：技术验证，而非表演性同意。

#### `skills/using-git-worktrees/` — 使用 Git Worktree

在需要与当前工作空间隔离的情况下启动功能工作。

核心原则：系统化的目录选择 + 安全验证 = 可靠的隔离。

#### `skills/finishing-a-development-branch/` — 完成开发分支

在实现完成、所有测试通过后，指导如何整合工作。

流程：验证测试 → 确定基础分支 → 呈现 4 个选项 → 执行选择 → 清理 worktree

---

### 3.4 元技能

#### `skills/using-superpowers/` — 技能系统入口

这是整个 Superpowers 的**元技能**，在每次新会话启动时自动加载。

核心规则：
- **强制规则**：只要有 1% 的可能性某个技能适用，就必须调用它检查
- **优先级**：用户指令 > Superpowers 技能 > 默认系统提示
- **工具适配**：技能使用 Claude Code 工具名，但需适配 Codex/Gemini CLI 的等价工具
- **技能类型**：Rigid（严格执行，如 TDD）vs Flexible（灵活适应）

包含：
- `SKILL.md` — 元技能主文件
- `references/codex-tools.md` — Codex 工具映射表
- `references/gemini-tools.md` — Gemini CLI 工具映射表

#### `skills/writing-skills/` — 编写技能

创建新技能、编辑现有技能或部署前验证技能。

核心原则：**编写技能就是 TDD 应用于流程文档**。

RED-GREEN-REFACTOR 映射：
- 测试用例 → 压力场景 + 子代理
- 生产代码 → 技能文档（SKILL.md）
- 测试失败 → 代理无技能时违反规则
- 测试通过 → 代理有技能时遵守

包含：
- `SKILL.md` — 编写技能主文件
- `anthropic-best-practices.md` — Anthropic 官方技能编写最佳实践

---

## 4. agents/ 目录 — 代理定义

### `agents/code-reviewer.md` — 代码审查代理

定义为**资深代码审查者**，在重大步骤完成后对照原始计划审查：
- 计划对齐分析
- 代码质量评估
- 架构与设计审查
- 文档与标准检查
- 问题分类（Critical / Important / Minor）
- 沟通协议

---

## 5. hooks/ 目录 — 会话钩子

hooks 目录包含了会话启动钩子，用于在每次新会话开始时自动注入 Superpowers 技能系统的引导内容。这是实现"每次会话自动加载技能系统"的核心机制。

### 核心机制

```
用户启动 Claude Code / Cursor
    ↓
触发 SessionStart / sessionStart 钩子
    ↓
执行 hooks/session-start 脚本
    ↓
读取 using-superpowers/SKILL.md 内容
    ↓
JSON 转义处理
    ↓
根据平台输出到额外上下文
    ↓
会话开始时自动拥有 superpowers 技能系统
```

### 文件详解

#### `hooks/session-start` — 会话启动钩子（核心）

这是一个 **bash 脚本**，是整个钩子系统的核心。

**功能流程：**

1. **检查遗留技能目录**
   - 检查 `~/.config/superpowers/skills` 是否存在
   - 如果存在，生成警告消息告知用户迁移到新位置

2. **读取 using-superpowers 内容**
   - 读取 `skills/using-superpowers/SKILL.md` 的完整内容
   - 这是每次会话都会自动加载的元技能

3. **JSON 转义处理**
   - 使用纯 bash 参数替换进行 JSON 转义（避免外部命令依赖）
   - 转义：反斜杠、双引号、换行、回车、制表符
   - 比字符循环快几个数量级

4. **构建注入内容**
   - 包装在 `<EXTREMELY_IMPORTANT>` 标签中
   - 包含完整的 using-superpowers 技能内容

5. **平台适配输出**
   - **Cursor**（检测 `CURSOR_PLUGIN_ROOT`）：输出 `additional_context`
   - **Claude Code**（检测 `CLAUDE_PLUGIN_ROOT`）：输出 `hookSpecificOutput.additionalContext`
   - **其他平台**：回退到 `additional_context`

**关键设计决策：**
- 使用 `printf` 而非 heredoc（避免 bash 5.3+ 的变量展开挂起 bug）
- 必须只输出当前平台消费的字段，避免重复注入
- Exit 0 确保钩子失败不会中断会话

#### `hooks/hooks.json` — Claude Code 钩子配置

```json
{
  "version": 1,
  "hooks": {
    "sessionStart": [
      {
        "command": "./hooks/session-start"
      }
    ]
  }
}
```

- 定义 `sessionStart` 钩子触发 `session-start` 脚本
- 版本 1 格式

#### `hooks/hooks-cursor.json` — Cursor 钩子配置

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|clear|compact",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
            "async": false
          }
        ]
      }
    ]
  }
}
```

- 使用 **matcher** 过滤触发条件：仅在 `startup`、`clear`、`compact` 时触发
- 使用 **run-hook.cmd** 跨平台包装器
- 设置 `async: false` 确保钩子在会话开始前完成

#### `hooks/run-hook.cmd` — 跨平台 Polyglot 包装器

这是一个巧妙的**双语言脚本**，同时兼容 Windows CMD 和 Unix bash。

**工作原理：**

```
Windows (cmd.exe) 视角：
┌─────────────────────────────────────────────┐
│ CMDBLOCK ~ (ignored by CMD)                │
│ @echo off                                   │
│ REM Windows batch commands                  │
│ REM Finds Git Bash and calls the script     │
│ exit /b 0                                  │
└─────────────────────────────────────────────┘
Unix (bash) 视角：
┌─────────────────────────────────────────────┐
│ : << 'CMDBLOCK'   (no-op in bash)          │
│ # Unix: run the named script directly       │
│ exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}"    │
└─────────────────────────────────────────────┘
```

**Windows 流程：**
1. CMD.exe 忽略 `: << 'CMDBLOCK'`（注释）
2. 执行 `@echo off` 和 `REM` 注释
3. 执行 `if "%~1"==""` 等 batch 命令
4. 查找 Git Bash（`C:\Program Files\Git\bin\bash.exe`）
5. 调用实际的 shell 脚本

**Unix 流程：**
1. Bash 解释 `: << 'CMDBLOCK'` 为 heredoc 开始（忽略内容）
2. 执行 CMDBLOCK 后的 bash 代码
3. 直接执行 shell 脚本

**为什么需要这个？**
- Claude Code 的 Windows 版本会自动检测 `.sh` 扩展名并预置 `bash`
- 使用无扩展名文件名 + 包装器避免干扰
- 确保 Windows 上 Claude Code 能正确调用 bash 脚本

### 遗留警告机制

```bash
legacy_skills_dir="${HOME}/.config/superpowers/skills"
if [ -d "$legacy_skills_dir" ]; then
    warning_message="\n\n<important-reminder>WARNING: ..."
fi
```

当检测到旧版安装时，会在会话首次回复时提醒用户迁移自定义技能到 `~/.claude/skills`。

---

## 6. docs/ 目录 — 文档

| 文件 | 说明 |
|------|------|
| `README.codex.md` | Codex 安装使用指南 |
| `README.opencode.md` | OpenCode 安装使用完整指南 |
| `testing.md` | Superpowers 技能测试指南 |
| `windows/polyglot-hooks.md` | Windows 跨平台钩子指南 |
| `plans/` | 项目内部实施计划 |
| `superpowers/plans/` | Superpowers 自身开发计划 |
| `superpowers/specs/` | Superpowers 自身设计规范 |

---

## 7. openspec/ 目录 — 项目设计规范

OpenSpec 是一个规范驱动的 AI 项目框架配置。

### 变更提案

#### `add-chinese-translation-rules/` — 中文翻译规范

**命名规范**：翻译文件使用 `{原文件名}.zh-CN.md` 后缀

**格式规范**：
- YAML frontmatter: 英文字段在前，中文译文在下一行
- 标题: 英文在前，中文在下一行（同级）
- 列表项: 英文在前，**中文翻译**加粗在下一行，缩进 2 空格
- 数字列表: 英文保留编号，中文去掉编号用缩进代替
- 代码块注释: 英文在前，中文在后
- 表格: 英文行 + 中文行交替

---

## 8. tests/ 目录 — 测试套件

| 目录 | 说明 |
|------|------|
| `tests/claude-code/` | Claude Code 集成测试 |
| `tests/brainstorm-server/` | Brainstorm 服务器测试 |
| `tests/opencode/` | OpenCode 测试 |
| `tests/explicit-skill-requests/` | 显式技能请求测试 |
| `tests/skill-triggering/` | 技能触发测试 |
| `tests/subagent-driven-dev/` | 子代理驱动开发测试项目 |

---

## 9. 工作流程总览

```
用户启动会话
    ↓
[session-start hook] 注入 bootstrap 内容
    ↓
[using-superpowers skill] 检查技能适用性
    ↓
如果需要创意工作 → [brainstorming skill]
    ↓ (设计批准后)
[using-git-worktrees skill] 创建隔离工作空间
    ↓
[writing-plans skill] 编写实现计划
    ↓ (计划审查循环)
    ↓
[subagent-driven-development skill] 推荐
    OR [executing-plans skill]
    ↓ (每个任务)
  派发实施者 subagent
    ↓
  spec 合规性审查
    ↓
  代码质量审查
    ↓
[finishing-a-development-branch skill] 验证测试并呈现整合选项
```

---

## 10. 平台支持矩阵

| 平台 | 技能 | Subagent | 钩子 | 特殊配置 |
|------|------|----------|------|----------|
| Claude Code | 原生 | 原生 | 原生 hooks | `.claude-plugin/` |
| Cursor | 原生 | 原生 | hooks-cursor.json | `.cursor-plugin/` |
| Codex | 通过符号链接 | 通过 `multi_agent` 配置 | N/A | `.codex/INSTALL.md` |
| OpenCode | 通过插件 | `@mention` 语法 | N/A | `.opencode/plugins/` |
| Gemini CLI | 通过 `GEMINI.md` | **不支持** | N/A | `gemini-extension.json` |

---

## 11. 技能速查表

| 技能 | 触发场景 | 铁律 |
|------|----------|------|
| `brainstorming` | 创造性工作之前 | 设计批准前不写代码 |
| `writing-plans` | 有规范的多步骤任务 | 执行者零背景知识 |
| `executing-plans` | 执行书面计划 | 带审查检查点 |
| `subagent-driven-development` | 推荐的计划执行方式 | 两阶段审查 |
| `test-driven-development` | 实现功能或修复 bug | 无失败测试不写代码 |
| `systematic-debugging` | 遇到 bug 或测试失败 | 无根本原因不修复 |
| `verification-before-completion` | 声称工作完成 | 无新鲜验证证据不声称 |
| `requesting-code-review` | 合并前验证 | 按严重程度分类问题 |
| `receiving-code-review` | 收到审查反馈 | 技术验证，非表演性同意 |
| `using-git-worktrees` | 需要隔离工作空间 | 目录选择 + 安全验证 |
| `finishing-a-development-branch` | 开发完成，准备整合 | 验证后呈现整合选项 |
| `dispatching-parallel-agents` | 2+ 独立任务 | 每个问题域一个代理 |
| `writing-skills` | 创建/编辑技能 | TDD 应用于流程文档 |
| `using-superpowers` | 每次会话开始 | 1% 可能性就调用技能 |

---

## 12. 已翻译的技能文件

以下技能已创建中文翻译文件 `SKILL.zh-CN.md`：

- ✅ brainstorming
- ✅ dispatching-parallel-agents
- ✅ executing-plans
- ✅ finishing-a-development-branch
- ✅ receiving-code-review
- ✅ requesting-code-review
- ✅ subagent-driven-development
- ✅ systematic-debugging
- ✅ test-driven-development
- ✅ using-git-worktrees
- ✅ using-superpowers
- ✅ verification-before-completion
- ✅ writing-plans
- ✅ writing-skills

---

*本文档由 AI 生成，最后更新：2026-03-19*
