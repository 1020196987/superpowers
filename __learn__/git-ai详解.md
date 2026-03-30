# git-ai 项目详解

## 项目概述

**git-ai** 是一个开源的 Git 扩展（Rust 语言开发），用于追踪代码仓库中 AI 生成的代码，将每行代码与其对应的 AI Agent、模型和会话记录关联起来。

- **GitHub**: https://github.com/git-ai-project/git-ai
- **官网**: https://usegitai.com
- **Star**: 1,438 | **Fork**: 113
- **许可证**: Apache 2.0

### 核心价值

> 一旦安装，它自动将每行 AI 生成的代码链接到生成它的 Agent、模型和会话记录——让你永远不会丢失代码背后的意图、需求和架构决策。

---

## 核心功能

### 1. AI Blame

`git-ai blame` 是 `git blame` 的替代品，显示每行代码的 AI 生成者（Agent、模型、Session）。

```bash
git-ai blame /src/log_fmt/authorship_log.rs
```

```bash
cb832b7 (Aidan Cunniffe      2025-12-13 08:16:29 -0500  133) pub fn execute_diff(
cb832b7 (Aidan Cunniffe      2025-12-13 08:16:29 -0500  134)     repo: &Repository,
cb832b7 (Aidan Cunniffe      2025-12-13 08:16:29 -0500  135)     spec: DiffSpec,
cb832b7 (Aidan Cunniffe      2025-12-13 08:16:29 -0500  136)     format: DiffFormat,
cb832b7 (Aidan Cunniffe      2025-12-13 08:16:29 -0500  137) ) -> Result<String, GitAiError> {
fe2c4c8 (claude [session_id] 2025-12-02 19:25:13 -0500  138)     // Resolve commits to get from/to SHAs
fe2c4c8 (claude [session_id] 2025-12-02 19:25:13 -0500  139)     let (from_commit, to_commit) = match spec {
```

### 2. Commit 时自动标注

在 `git commit` 输出中显示 human/AI 代码比例：

```
[hooks-doctor 0afe44b2] wsl compat check
 2 files changed, 81 insertions(+), 3 deletions(-)
you  ██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ ai
     6%             mixed   2%             92%
```

### 3. 跨 Agent 可观测性

通过 `git-ai stats` 统计不同 Agent 的代码贡献量：

```bash
git-ai stats --json
```

```json
{
  "human_additions": 28,
  "mixed_additions": 5,
  "ai_additions": 76,
  "ai_accepted": 47,
  "total_ai_additions": 120,
  "total_ai_deletions": 34,
  "time_waiting_for_ai": 240,
  "tool_model_breakdown": {
    "claude_code/claude-sonnet-4-5-20250929": {
      "ai_additions": 76,
      "ai_accepted": 47,
      "total_ai_additions": 120,
      "total_ai_deletions": 34
    }
  }
}
```

### 4. `/ask` 技能

可以直接向生成该代码的 Agent 提问，理解代码背后的意图：

```
/ask Why didn't we use the SDK here?
```

---

## 支持的 Agent

Claude Code、Codex、Cursor、Windsurf、Copilot、Continue、Gemini、Junie、Rovo Dev、Amp 等主流 AI 编程工具。

---

## 工作原理

### 整体流程

```
用户编辑代码
    ↓
Agent 执行编辑（通过 pre/post edit hooks 报告写的行）
    ↓
git-ai 在 .git/ai/ 存储 checkpoints（小diff）
    ↓
git commit 时
    ↓
处理所有 checkpoints → 生成 Authorship Log
    ↓
通过 Git Note (refs/notes/ai) 附加到 commit
```

### 1. Checkpoint 记录

Agent 通过 **pre/post edit hooks** 报告它们写了哪些代码。git-ai 在 `.git/ai/` 目录下存储每个编辑的小 diff（称为 checkpoint），记录该变更是否由 AI 生成。

### 2. Authorship Log 生成（Commit 时）

提交时，git-ai 处理所有 checkpoints，生成 **Authorship Log**，将行范围映射到 Agent Session。

### 3. Git Note 存储

git-ai 使用 **Git Notes**（`refs/notes/ai` 命名空间）存储 AI 作者信息，不修改 commit 历史。

#### Git Notes 简介

Git Notes 是 Git 的一个功能，允许在不修改 commit 的情况下为 commit 添加额外的元数据信息。

| 特性 | 说明 |
|------|------|
| **不修改 commit** | Notes 存储在独立的引用空间，不影响原有 commit 历史 |
| **独立引用** | 存储在 `refs/notes/` 命名空间 |
| **可共享** | 可以推送到远程，和其他 Git 数据一起同步 |
| **灵活** | 每个 commit 可以有 0 个或 1 个 note |

```bash
# 添加 note
git notes add -m "message" <commit-sha>

# 查看 note
git notes show <commit-sha>

# 列出所有 notes 引用
git notes list
```

### 4. Authorship Log 格式

Git Note 存储的 Authorship Log 格式示例：

```
hooks/post_clone_hook.rs
  a1b2c3d4e5f6a7b8 6-8
  c9d0e1f2a3b4c5d6 16,21,25
---
{
  "schema_version": "authorship/3.0.0",
  "base_commit_sha": "f4a8b2c...",
  "prompts": {
    "a1b2c3d4e5f6a7b8": {
      "agent_id": { "tool": "copilot", "model": "codex-5.2" },
      "human_author": "Alice Person <alice@example.com>",
      "messages_url": "https://your-prompt-store.dev/cas/a1b2c3d4..."
    },
    "c9d0e1f2a3b4c5d6": {
      "agent_id": { "tool": "cursor", "model": "sonnet-4.5" },
      "human_author": "Jeff Coder <jeff@example.com>",
      "messages_url": "https://your-prompt-store.dev/cas/c9d0e1f2..."
    }
  }
}
```

格式规范定义在 [Git AI Standard v3.0.0](https://github.com/git-ai-project/git-ai/blob/main/specs/git_ai_standard_v3.0.0.md)。

### 5. 历史变更追踪

git-ai 在以下操作时**透明地重写 Authorship Log**，保持 attribution 不丢失：

- Rebases
- Merges
- Squashes
- Cherry-picks
- Stash/pops
- Commit amends

---

## 设计哲学

| 原则 | 说明 |
|------|------|
| **无需改工作流** | 安装后即用，无需 per-repo 配置 |
| **不"检测"AI代码** | Agent 主动报告写的行，不靠猜测 |
| **Local-first** | 100% 离线可用，无需登录 |
| **Transcript 不进 Git** | 敏感信息留在本地或云端 prompt store |
| **Git 原生** | 利用 Git Notes，不污染 commit 历史 |

---

## 安装方式

### Mac / Linux / Windows (WSL)

```bash
curl -sSL https://usegitai.com/install.sh | bash
```

### Windows (非 WSL)

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "irm https://usegitai.com/install.ps1 | iex"
```

---

## 开放标准

git-ai 遵循 [Git AI Standard v3.0.0](https://github.com/git-ai-project/git-ai/blob/main/specs/git_ai_standard_v3.0.0.md)，这是一个开放标准，其他项目也可以实现兼容的 AI 代码追踪方案。

### 标准核心要点

- **命名空间**: 使用 `refs/notes/ai`（不是默认的 `refs/notes/commits`）
- **Schema Version**: `authorship/3.0.0`
- **Session Hash**: 使用 SHA-256 of `{tool}:{conversation_id}` 的前 16 位十六进制字符
- **Line Range**: 1-indexed，逗号分隔的单个行号和范围

---

*本文档由 AI 生成，最后更新：2026-03-30*
