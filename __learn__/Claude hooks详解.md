# CLAUDE_PLUGIN_ROOT 变量说明

## 什么是 CLAUDE_PLUGIN_ROOT？

`CLAUDE_PLUGIN_ROOT` 是 Claude Code 在执行 hook 时**自动注入的环境变量**，指向当前执行的 hook 所在插件的目录路径。

# CLAUDE_PLUGIN_ROOT 变量说明

## 什么是 CLAUDE_PLUGIN_ROOT？

`CLAUDE_PLUGIN_ROOT` 是 Claude Code 在执行 hook 时**自动注入的环境变量**，指向当前执行的 hook 所在插件的目录路径。

## Hook 加载机制

Claude Code 有两种 hook 配置方式：

### 1. 插件 Hook（推荐方式）

在插件目录下的 `hooks/hooks.json` 中定义：

```
插件目录/
└── hooks/
    └── hooks.json    ← Claude Code 启动时自动扫描发现
```

**加载流程：**
1. Claude Code 启动时扫描所有已安装插件
2. 检查每个插件是否存在 `hooks/hooks.json`
3. 将发现的 hooks 加载到当前会话
4. **插件必须被启用**才能生效

### 2. 用户级 Hook

直接在 `~/.claude/settings.json` 中配置：

```json
{
  "PreToolUse": [...],
  "Stop": [...]
}
```

**特点：**
- 无需包裹层，直接在顶层定义事件
- 无 `description` 字段
- 立即生效（下次会话）

## 大模型如何知道有 hooks？

**不是大模型"知道"，是 Claude Code 运行时框架在拦截事件时执行 hook 逻辑：**

```
用户发送消息
    ↓
Claude Code 拦截 [UserPromptSubmit hook 触发]
    ↓
执行 hook 配置的逻辑（prompt 或 command）
    ↓
根据 hook 输出决定如何处理
    ↓
大模型收到的是经过 hook 处理后的上下文
```

Hook 在**框架层面**执行，模型本身不知道有 hooks 存在。

## 生效条件

| 条件 | 说明 |
|------|------|
| 插件已启用 | 在 `settings.json` 的 `plugins` 列表中 |
| 会话启动 | Hook 在会话开始时加载 |
| 重启生效 | 修改 hooks 后需重启 Claude Code |

## 配置位置对比

| 方式 | 配置文件 | 格式 |
|------|---------|------|
| 插件 Hook | `插件/hooks/hooks.json` | `{"hooks": {...}}` |
| 用户 Hook | `~/.claude/settings.json` | 直接顶层配置 |

## 常见误区

- ❌ 不是用户手动定义的
- ❌ 不是写在某处配置文件的
- ✅ 由 Claude Code 运行时自动注入到 command hooks

## 可用场景

在所有 **command hooks** 中可直接使用：

```json
{
  "type": "command",
  "command": "bash $CLAUDE_PLUGIN_ROOT/scripts/validate.sh"
}
```

## 实际值示例

假设你正在使用 `everything-claude-code` 插件：

```
CLAUDE_PLUGIN_ROOT = /Users/pengchengming/.claude/plugins/marketplaces/everything-claude-code
```

不同插件的 hook 运行时，`CLAUDE_PLUGIN_ROOT` 会指向各自插件目录。

## 相关的环境变量

| 变量 | 说明 |
|------|------|
| `$CLAUDE_PROJECT_DIR` | 项目根目录路径 |
| `$CLAUDE_PLUGIN_ROOT` | 插件目录路径 |
| `$CLAUDE_ENV_FILE` | SessionStart 专用：持久化环境变量的文件 |
| `$CLAUDE_CODE_REMOTE` | 是否运行在远程上下文 |

## 为什么使用它？

**可移植性**：使用 `$CLAUDE_PLUGIN_ROOT` 而不是硬编码绝对路径，让 hook 脚本能在不同环境下复用。

```bash
# ✅ 好：可移植
bash $CLAUDE_PLUGIN_ROOT/scripts/validate.sh

# ❌ 差：硬编码路径
bash /Users/pengchengming/.claude/plugins/cache/claude-plugins-official/plugin-dev/55b58ec6e564/scripts/validate.sh
```
