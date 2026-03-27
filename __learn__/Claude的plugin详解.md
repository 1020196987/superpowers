# Claude Code Plugins 目录结构与加载机制

## 目录结构

```
~/.claude/plugins/
├── marketplaces/          # Marketplace 插件源码（只读）
│   ├── claude-plugins-official/
│   ├── anthropic-agent-skills/
│   ├── everything-claude-code/
│   └── superpowers-marketplace/
├── cache/                # 实际运行的插件实例
│   ├── claude-plugins-official/
│   ├── anthropic-agent-skills/
│   ├── everything-claude-code/
│   └── superpowers-marketplace/
├── installed_plugins.json   # 已安装插件清单
├── known_marketplaces.json  # 已知的 Marketplace 列表
├── blocklist.json           # 插件阻止列表
└── data/                   # 插件数据目录
```

## 两个目录的区别

| 目录 | 作用 | 内容 |
|------|------|------|
| `marketplaces/` | 存放从 marketplace 下载的原始插件代码 | 插件源码，可更新时拉取新版本 |
| `cache/` | 实际运行的插件实例 | 根据 installed_plugins.json 确定的版本 |

**为什么分离？**
- `marketplaces/` 是原始仓库副本
- `cache/` 是根据 `installed_plugins.json` 中指定版本运行的实例
- 方便版本管理和回滚

## 加载机制

### 1. 读取已安装插件清单

Claude Code 启动时首先读取 `installed_plugins.json`：

```json
{
  "version": 2,
  "plugins": {
    "everything-claude-code@everything-claude-code": [
      {
        "scope": "user",
        "installPath": "/Users/pengchengming/.claude/plugins/cache/everything-claude-code/everything-claude-code/1.8.0",
        "version": "1.8.0"
      }
    ]
  }
}
```

### 2. 根据 installPath 加载插件

1. 读取 `installed_plugins.json` 获取所有已安装插件
2. 根据每个插件的 `installPath` 找到实际运行的插件目录
3. 加载插件的 `plugin.json` 清单文件
4. 扫描并加载插件的 hooks（`hooks/hooks.json`）

### 3. Hook 发现流程

```
Claude Code 启动
    ↓
读取 installed_plugins.json
    ↓
遍历每个插件的 installPath
    ↓
读取插件目录的 plugin.json
    ↓
检查 hooks/hooks.json 是否存在
    ↓
加载并注册 hooks 到当前会话
```

## 插件清单文件详解

### plugin.json 位置

| 路径 | 说明 |
|------|------|
| `插件根目录/plugin.json` | 旧版/简单插件 |
| `插件根目录/.claude-plugin/plugin.json` | 新版插件（分离 npm 包结构） |

`everything-claude-code` 采用新版结构：
```
everything-claude-code/
├── package.json              # npm 包管理
├── .claude-plugin/
│   └── plugin.json           # Claude Code 插件配置
├── hooks/
├── agents/
└── skills/
```

### installed_plugins.json

记录所有已安装插件及其运行版本：

```json
{
  "plugins": {
    "插件名称@来源": [
      {
        "scope": "user",           // user 或 project 级别
        "installPath": "...",       // 实际运行路径
        "version": "1.8.0",        // 版本号
        "installedAt": "时间戳",    // 安装时间
        "lastUpdated": "时间戳",    // 最后更新时间
        "gitCommitSha": "..."      // Git 提交 SHA
      }
    ]
  }
}
```

### plugin.json（插件内部）

每个插件根目录必须有 `plugin.json`：

```json
{
  "name": "everything-claude-code",
  "version": "1.8.0",
  "description": "...",
  "hooks": {
    "PreToolUse": [...]
  }
}
```

## 插件目录结构示例

```
everything-claude-code/
├── plugin.json              # 插件清单
├── hooks/
│   └── hooks.json            # Hook 配置
├── agents/                  # 子代理
├── skills/                  # 技能
├── commands/                # 命令
└── README.md
```

## 加载条件

| 条件 | 说明 |
|------|------|
| 插件在 `installed_plugins.json` 中 | 已安装且启用 |
| `plugin.json` 存在且有效 | 插件清单完整 |
| `hooks/hooks.json` 存在 | 有 hook 配置 |

## 查看当前加载的插件

```bash
# 方式1：查看 installed_plugins.json
cat ~/.claude/plugins/installed_plugins.json

# 方式2：在 Claude Code 中使用 /plugins 命令（如果支持）

# 方式3：使用 --debug 模式启动
claude --debug
```

## 插件来源标识

插件名格式：`插件名@来源`

| 来源格式 | 说明 |
|---------|------|
| `@claude-plugins-official` | Anthropic 官方插件 |
| `@anthropic-agent-skills` | Anthropic Agent Skills |
| `@everything-claude-code` | Marketplace 插件（同名） |
| `@superpowers-marketplace` | 第三方 Marketplace |

## 缓存更新机制

当插件更新时：
1. 从 marketplace 拉取新版本到 `marketplaces/`
2. 更新 `installed_plugins.json` 中的 `installPath` 和版本
3. 下次启动时加载新版本
