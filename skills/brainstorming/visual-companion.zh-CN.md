# Visual Companion Guide
# 视觉伴侣指南

Browser-based visual brainstorming companion for showing mockups, diagrams, and options.
用于展示原型、图表和选项的基于浏览器的视觉头脑风暴伴侣。

## When to Use
## 使用场景

Decide per-question, not per-session. The test: **would the user understand this better by seeing it than reading it?**
逐问题决定，而非逐会话决定。判断标准：**用户通过观看比阅读更能理解吗？**

**Use the browser** when the content itself is visual:
**使用浏览器**当内容本身是视觉化的：

- **UI mockups** — wireframes, layouts, navigation structures, component designs
  **UI 原型** — 线框图、布局、导航结构、组件设计
- **Architecture diagrams** — system components, data flow, relationship maps
  **架构图** — 系统组件、数据流、关系图
- **Side-by-side visual comparisons** — comparing two layouts, two color schemes, two design directions
  **并排视觉比较** — 比较两种布局、两种配色方案、两种设计方向
- **Design polish** — when the question is about look and feel, spacing, visual hierarchy
  **设计优化** — 当问题涉及外观和感觉、间距、视觉层次时
- **Spatial relationships** — state machines, flowcharts, entity relationships rendered as diagrams
  **空间关系** — 状态机、流程图、以图表形式呈现的实体关系

**Use the terminal** when the content is text or tabular:
**使用终端**当内容是文本或表格时：

- **Requirements and scope questions** — "what does X mean?", "which features are in scope?"
  **需求和范围问题** — "X 是什么意思？"，"哪些功能在范围内？"
- **Conceptual A/B/C choices** — picking between approaches described in words
  **概念性的 A/B/C 选择** — 选择用文字描述的方法
- **Tradeoff lists** — pros/cons, comparison tables
  **权衡列表** — 优点/缺点、比较表
- **Technical decisions** — API design, data modeling, architectural approach selection
  **技术决策** — API 设计、数据建模、架构方法选择
- **Clarifying questions** — anything where the answer is words, not a visual preference
  **澄清问题** — 答案用文字而非视觉偏好的任何内容

A question *about* a UI topic is not automatically a visual question. "What kind of wizard do you want?" is conceptual — use the terminal. "Which of these wizard layouts feels right?" is visual — use the browser.
关于 UI 主题的问题不自动是视觉问题。"你想要什么样的向导？"是概念性的——使用终端。"哪个向导布局感觉正确？"是视觉的——使用浏览器。

## How It Works
## 工作原理

The server watches a directory for HTML files and serves the newest one to the browser. You write HTML content, the user sees it in their browser and can click to select options. Selections are recorded to a `.events` file that you read on your next turn.
服务器监视目录中的 HTML 文件，并将最新的文件提供给浏览器。你写入 HTML 内容，用户在浏览器中看到并可以点击选择选项。选择被记录到 `.events` 文件中，你在下一轮读取它。

**Content fragments vs full documents:** If your HTML file starts with `<!DOCTYPE` or `<html`, the server serves it as-is (just injects the helper script). Otherwise, the server automatically wraps your content in the frame template — adding the header, CSS theme, selection indicator, and all interactive infrastructure. **Write content fragments by default.** Only write full documents when you need complete control over the page.
**内容片段 vs 完整文档：** 如果你的 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器按原样提供（只是注入辅助脚本）。否则，服务器自动将你的内容包装在框架模板中——添加标题、CSS 主题、选择指示器和所有交互基础设施。**默认写内容片段。** 只有当你需要完全控制页面时才写完整文档。

## Starting a Session
## 启动会话

```bash
# Start server with persistence (mockups saved to project)
# 启动服务器并持久化（原型保存到项目）
scripts/start-server.sh --project-dir /path/to/project

# Returns: {"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000"}
```

Save `screen_dir` from the response. Tell user to open the URL.
从响应中保存 `screen_dir`。告诉用户打开 URL。

**Finding connection info:** The server writes its startup JSON to `$SCREEN_DIR/.server-info`. If you launched the server in the background and didn't capture stdout, read that file to get the URL and port. When using `--project-dir`, check `<project>/.superpowers/brainstorm/` for the session directory.
**查找连接信息：** 服务器将启动 JSON 写入 `$SCREEN_DIR/.server-info`。如果你在后台启动了服务器且没有捕获 stdout，读取该文件以获取 URL 和端口。使用 `--project-dir` 时，检查 `<project>/.superpowers/brainstorm/` 中的会话目录。

**Note:** Pass the project root as `--project-dir` so mockups persist in `.superpowers/brainstorm/` and survive server restarts. Without it, files go to `/tmp` and get cleaned up. Remind the user to add `.superpowers/` to `.gitignore` if it's not already there.
**注意：** 将项目根目录作为 `--project-dir` 传递，这样原型会持久化在 `.superpowers/brainstorm/` 中并在服务器重启后保留。没有它，文件会进入 `/tmp` 并被清理。提醒用户将 `.superpowers/` 添加到 `.gitignore`（如果还没有）。

**Launching the server by platform:**
**按平台启动服务器：**

**Claude Code (macOS / Linux):**
```bash
# Default mode works — the script backgrounds the server itself
# 默认模式可用——脚本自己将服务器置于后台
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code (Windows):**
```bash
# Windows auto-detects and uses foreground mode, which blocks the tool call.
# Windows 自动检测并使用前台模式，这会阻塞工具调用。
# Use run_in_background: true on the Bash tool call so the server survives
# 在 Bash 工具调用上使用 run_in_background: true，这样服务器可以在
# across conversation turns.
# 对话轮次之间保持运行。
scripts/start-server.sh --project-dir /path/to/project
```
When calling this via the Bash tool, set `run_in_background: true`. Then read `$SCREEN_DIR/.server-info` on the next turn to get the URL and port.
通过 Bash 工具调用时，设置 `run_in_background: true`。然后在下一轮读取 `$SCREEN_DIR/.server-info` 获取 URL 和端口。

**Codex:**
```bash
# Codex reaps background processes. The script auto-detects CODEX_CI and
# Codex 会回收后台进程。脚本自动检测 CODEX_CI 并
# switches to foreground mode. Run it normally — no extra flags needed.
# 切换到前台模式。正常运行——不需要额外标志。
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI:**
```bash
# Use --foreground and set is_background: true on your shell tool call
# 使用 --foreground 并在你的 shell 工具调用上设置 is_background: true
# so the process survives across turns
# 这样进程可以在轮次之间保持运行
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**Other environments:** The server must keep running in the background across conversation turns. If your environment reaps detached processes, use `--foreground` and launch the command with your platform's background execution mechanism.
**其他环境：** 服务器必须在对话轮次之间保持后台运行。如果你的环境会回收分离的进程，使用 `--foreground` 并用平台的后台执行机制启动命令。

If the URL is unreachable from your browser (common in remote/containerized setups), bind a non-loopback host:
如果你的浏览器无法访问 URL（远程/容器化设置中常见），绑定一个非回环主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

Use `--url-host` to control what hostname is printed in the returned URL JSON.
使用 `--url-host` 控制返回的 URL JSON 中打印的主机名。

## The Loop
## 循环

1. **Check server is alive**, then **write HTML** to a new file in `screen_dir`:
   **检查服务器存活**，然后**写入 HTML** 到 `screen_dir` 中的新文件：
   - Before each write, check that `$SCREEN_DIR/.server-info` exists. If it doesn't (or `.server-stopped` exists), the server has shut down — restart it with `start-server.sh` before continuing. The server auto-exits after 30 minutes of inactivity.
     **在每次写入前，检查 `$SCREEN_DIR/.server-info` 是否存在。如果不存在（或存在 `.server-stopped`），服务器已关闭——在继续之前用 `start-server.sh` 重启它。服务器在 30 分钟不活动后自动退出。**
   - Use semantic filenames: `platform.html`, `visual-style.html`, `layout.html`
     **使用语义化文件名：** `platform.html`、`visual-style.html`、`layout.html`
   - **Never reuse filenames** — each screen gets a fresh file
     **永远不要重用文件名** — 每个屏幕获得一个新文件
   - Use Write tool — **never use cat/heredoc** (dumps noise into terminal)
     **使用 Write 工具** — **永远不要用 cat/heredoc**（会在终端中产生噪音）
   - Server automatically serves the newest file
     **服务器自动提供最新文件**

2. **Tell user what to expect and end your turn:**
   **告诉用户预期内容并结束你的回合：**
   - Remind them of the URL (every step, not just first)
     **提醒他们 URL（每步都要，不只是第一步）**
   - Give a brief text summary of what's on screen (e.g., "Showing 3 layout options for the homepage")
     **简要文字总结屏幕上的内容（例如，"显示首页的 3 个布局选项"）**
   - Ask them to respond in the terminal: "Take a look and let me know what you think. Click to select an option if you'd like."
     **请他们在终端中回复："看看并告诉我你的想法。如果愿意，点击选择一个选项。"**

3. **On your next turn** — after the user responds in the terminal:
   **在你下一轮** — 用户在终端中回复后：
   - Read `$SCREEN_DIR/.events` if it exists — this contains the user's browser interactions (clicks, selections) as JSON lines
     **如果存在，读取 `$SCREEN_DIR/.events`** — 这包含用户的浏览器交互（点击、选择）作为 JSON 行
   - Merge with the user's terminal text to get the full picture
     **与用户的终端文本合并以获得完整情况**
   - The terminal message is the primary feedback; `.events` provides structured interaction data
     **终端消息是主要反馈；`.events` 提供结构化的交互数据**

4. **Iterate or advance** — if feedback changes current screen, write a new file (e.g., `layout-v2.html`). Only move to the next question when the current step is validated.
   **迭代或推进** — 如果反馈改变了当前屏幕，写一个新文件（例如 `layout-v2.html`）。只有当当前步骤被验证后才进入下一步。

5. **Unload when returning to terminal** — when the next step doesn't need the browser (e.g., a clarifying question, a tradeoff discussion), push a waiting screen to clear the stale content:
   **返回终端时卸载** — 当下一步不需要浏览器时（例如澄清问题、权衡讨论），推送一个等待屏幕以清除过时内容：

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   This prevents the user from staring at a resolved choice while the conversation has moved on. When the next visual question comes up, push a new content file as usual.
   **这可以防止用户在对话已经进行时盯着已解决的选择。**当下一个视觉问题出现时，照常推送新的内容文件。

6. Repeat until done.
   **重复直到完成。**

## Writing Content Fragments
## 编写内容片段

Write just the content that goes inside the page. The server wraps it in the frame template automatically (header, theme CSS, selection indicator, and all interactive infrastructure).
只写放入页面内部的内容。服务器自动将其包装在框架模板中（标题、主题 CSS、选择指示器和所有交互基础设施）。

**Minimal example:**
**最小示例：**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

That's it. No `<html>`, no CSS, no `<script>` tags needed. The server provides all of that.
**就这样。**不需要 `<html>`、CSS 或 `<script>` 标签。服务器提供所有这些。

## CSS Classes Available
## 可用的 CSS 类

The frame template provides these CSS classes for your content:
框架模板为你的内容提供这些 CSS 类：

### Options (A/B/C choices)
### 选项（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**Multi-select:** Add `data-multiselect` to the container to let users select multiple options. Each click toggles the item. The indicator bar shows the count.
**多选：** 在容器上添加 `data-multiselect` 以允许用户选择多个选项。每次点击切换项目。指示器栏显示计数。

```html
<div class="options" data-multiselect>
  <!-- same option markup — users can select/deselect multiple -->
</div>
```

### Cards (visual designs)
### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### Mockup container
### 原型容器

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### Split view (side-by-side)
### 分屏视图（并排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### Pros/Cons
### 优点/缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### Mock elements (wireframe building blocks)
### 模拟元素（线框构建块）

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### Typography and sections
### 排版和章节

- `h2` — page title
  **页面标题**
- `h3` — section heading
  **章节标题**
- `.subtitle` — secondary text below title
  **标题下方的副文本**
- `.section` — content block with bottom margin
  **带底部边距的内容块**
- `.label` — small uppercase label text
  **小写大写标签文本**

## Browser Events Format
## 浏览器事件格式

When the user clicks options in the browser, their interactions are recorded to `$SCREEN_DIR/.events` (one JSON object per line). The file is cleared automatically when you push a new screen.
当用户在浏览器中点击选项时，他们的交互被记录到 `$SCREEN_DIR/.events`（每行一个 JSON 对象）。当你推送新屏幕时，文件会自动清除。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

The full event stream shows the user's exploration path — they may click multiple options before settling. The last `choice` event is typically the final selection, but the pattern of clicks can reveal hesitation or preferences worth asking about.
完整的事件流显示了用户的探索路径——他们可能在确定之前点击多个选项。最后的 `choice` 事件通常是最终选择，但点击模式可以揭示犹豫或值得询问的偏好。

If `.events` doesn't exist, the user didn't interact with the browser — use only their terminal text.
如果 `.events` 不存在，用户没有与浏览器交互——只使用他们的终端文本。

## Design Tips
## 设计提示

- **Scale fidelity to the question** — wireframes for layout, polish for polish questions
  **根据问题调整保真度** — 布局用线框图，优化问题用精细设计
- **Explain the question on each page** — "Which layout feels more professional?" not just "Pick one"
  **在每个页面上解释问题** — "哪种布局感觉更专业？"而不只是"选一个"
- **Iterate before advancing** — if feedback changes current screen, write a new version
  **推进前迭代** — 如果反馈改变了当前屏幕，写一个新版本
- **2-4 options max** per screen
  **每个屏幕最多 2-4 个选项**
- **Use real content when it matters** — for a photography portfolio, use actual images (Unsplash). Placeholder content obscures design issues.
  **重要时使用真实内容** — 对于摄影作品集，使用实际图片（Unsplash）。占位符内容会掩盖设计问题。
- **Keep mockups simple** — focus on layout and structure, not pixel-perfect design
  **保持原型简单** — 专注于布局和结构，而非像素级设计

## File Naming
## 文件命名

- Use semantic names: `platform.html`, `visual-style.html`, `layout.html`
  **使用语义化名称：** `platform.html`、`visual-style.html`、`layout.html`
- Never reuse filenames — each screen must be a new file
  **永远不要重用文件名** — 每个屏幕必须是一个新文件
- For iterations: append version suffix like `layout-v2.html`, `layout-v3.html`
  **对于迭代：** 附加版本后缀如 `layout-v2.html`、`layout-v3.html`
- Server serves newest file by modification time
  **服务器按修改时间提供最新文件**

## Cleaning Up
## 清理

```bash
scripts/stop-server.sh $SCREEN_DIR
```

If the session used `--project-dir`, mockup files persist in `.superpowers/brainstorm/` for later reference. Only `/tmp` sessions get deleted on stop.
如果会话使用了 `--project-dir`，原型文件会持久化在 `.superpowers/brainstorm/` 中供以后参考。只有 `/tmp` 会话会在停止时删除。

## Reference
## 参考

- Frame template (CSS reference): `scripts/frame-template.html`
- 框架模板（CSS 参考）：`scripts/frame-template.html`
- Helper script (client-side): `scripts/helper.js`
- 辅助脚本（客户端）：`scripts/helper.js`
