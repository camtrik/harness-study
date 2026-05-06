# 视觉伴侣指南

基于浏览器的视觉头脑风暴伴侣工具，用于展示 mockup、图表和选项。

## 何时使用

逐问题决策，而非逐会话决策。测试标准是：**用户通过观看比通过阅读能更好地理解这个内容吗？**

**使用浏览器** 当内容本身是视觉性质时：

- **UI mockup** — 线框图、布局、导航结构、组件设计
- **架构图** — 系统组件、数据流、关系图
- **并排视觉对比** — 对比两种布局、两种配色方案、两种设计方向
- **设计润色** — 当问题涉及外观和感觉、间距、视觉层次时
- **空间关系** — 状态机、流程图、实体关系图，以图表形式呈现

**使用终端** 当内容是文本或表格时：

- **需求和范围问题** — "X 是什么意思？"、"哪些功能在范围内？"
- **概念性的 A/B/C 选择** — 在文字描述的方案之间做出选择
- **权衡列表** — 优缺点、对比表格
- **技术决策** — API 设计、数据建模、架构方案选择
- **澄清问题** — 任何答案是用文字而非视觉偏好来表达的问题

*关于* UI 主题的问题不一定自动是视觉问题。"你想要什么样的向导？"是概念性的——使用终端。"这些向导布局中哪个感觉合适？"是视觉性的——使用浏览器。

## 工作原理

服务器监控一个目录中的 HTML 文件，并将最新的文件提供给浏览器。你将 HTML 内容写入 `screen_dir`，用户在浏览器中看到它，并可以点击选择选项。选择会记录到 `state_dir/events`，你在下一轮中读取这些内容。

**内容片段 vs 完整文档：** 如果你的 HTML 文件以 `<!DOCTYPE` 或 `<html` 开头，服务器按原样提供（仅注入辅助脚本）。否则，服务器会自动将你的内容包装在框架模板中——添加页头、CSS 主题、选择指示器和所有交互基础设施。**默认使用内容片段。** 只有当你需要完全控制页面时才编写完整文档。

## 启动会话

```bash
# 启动服务器并启用持久化（mockup 保存到项目中）
scripts/start-server.sh --project-dir /path/to/project

# 返回：{"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/content",
#           "state_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000/state"}
```

从响应中保存 `screen_dir` 和 `state_dir`。告诉用户打开 URL。

**查找连接信息：** 服务器将其启动 JSON 写入 `$STATE_DIR/server-info`。如果你在后台启动了服务器但没有捕获 stdout，读取该文件以获取 URL 和端口。使用 `--project-dir` 时，检查 `<project>/.superpowers/brainstorm/` 以查找会话目录。

**注意：** 将项目根目录作为 `--project-dir` 传入，以便 mockup 持久保存在 `.superpowers/brainstorm/` 中，并在服务器重启后继续存在。不加此参数，文件会存入 `/tmp` 并被清理。如果 `.superpowers/` 尚未在 `.gitignore` 中，提醒用户添加。

**按平台启动服务器：**

**Claude Code（macOS / Linux）：**
```bash
# 默认模式即可 — 脚本自身会在后台运行服务器
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code（Windows）：**
```bash
# Windows 自动检测并使用前台模式，这会阻塞工具调用。
# 在 Bash 工具调用上使用 run_in_background: true，使服务器在对话轮次间存活。
scripts/start-server.sh --project-dir /path/to/project
```
通过 Bash 工具调用此命令时，设置 `run_in_background: true`。然后在下一轮读取 `$STATE_DIR/server-info` 以获取 URL 和端口。

**Codex：**
```bash
# Codex 会收割后台进程。脚本自动检测 CODEX_CI 并切换到前台模式。
# 正常运行即可 — 无需额外标志。
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI：**
```bash
# 使用 --foreground 并在 shell 工具调用上设置 is_background: true，
# 使进程在轮次间存活
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**其他环境：** 服务器必须在对话轮次间持续在后台运行。如果你的环境会收割分离进程，使用 `--foreground` 并通过平台的 Background 执行机制启动命令。

如果从浏览器无法访问 URL（在远程/容器化环境中常见），绑定非回环主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

使用 `--url-host` 来控制返回的 URL JSON 中打印的主机名。

## 循环流程

1. **检查服务器是否存活**，然后**将 HTML 写入** `screen_dir` 中的新文件：
   - 每次写入之前，检查 `$STATE_DIR/server-info` 是否存在。如果不存在（或者 `$STATE_DIR/server-stopped` 存在），说明服务器已关闭——在继续之前用 `start-server.sh` 重启它。服务器在 30 分钟无活动后自动退出。
   - 使用语义化文件名：`platform.html`、`visual-style.html`、`layout.html`
   - **切勿重用文件名** — 每个画面使用新文件
   - 使用 Write 工具 — **切勿使用 cat/heredoc**（会向终端输出干扰信息）
   - 服务器自动提供最新文件

2. **告诉用户预期看到什么，然后结束你的轮次：**
   - 提醒他们 URL（每一步都提醒，不仅是第一次）
   - 给出屏幕上内容的简要文字摘要（例如，"显示首页的 3 种布局选项"）
   - 请他们在终端中回复："请查看并告诉我你的想法。如果想选择某个选项，可以点击。"

3. **在你的下一轮** — 用户在终端回复之后：
   - 读取 `$STATE_DIR/events`（如果存在）— 其中包含用户的浏览器交互（点击、选择），格式为 JSON 行
   - 与用户的终端文字合并以获取完整画面
   - 终端消息是主要反馈；`state_dir/events` 提供结构化的交互数据

4. **迭代或推进** — 如果反馈改变了当前画面，编写新文件（例如 `layout-v2.html`）。只有在当前步骤验证通过后才进入下一个问题。

5. **返回终端时卸载** — 当下一步不需要浏览器时（例如，澄清问题、权衡讨论），推送一个等待画面以清除过时内容：

   ```html
   <!-- 文件名：waiting.html（或 waiting-2.html 等） -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">继续在终端中进行...</p>
   </div>
   ```

   这可以防止用户在对话已经推进后还盯着一个已解决的选择。当下一个视觉问题出现时，照常推送新的内容文件。

6. 重复直到完成。

## 编写内容片段

只编写放在页面内部的内容。服务器会自动将其包装在框架模板中（页头、主题 CSS、选择指示器和所有交互基础设施）。

**最小示例：**

```html
<h2>哪种布局效果更好？</h2>
<p class="subtitle">考虑可读性和视觉层次</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>单栏</h3>
      <p>简洁、专注的阅读体验</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>双栏</h3>
      <p>侧边栏导航与主内容区</p>
    </div>
  </div>
</div>
```

就是这样。不需要 `<html>`、不需要 CSS、不需要 `<script>` 标签。服务器提供了所有这些。

## 可用的 CSS 类

框架模板为你的内容提供了以下 CSS 类：

### 选项（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>标题</h3>
      <p>描述</p>
    </div>
  </div>
</div>
```

**多选：** 在容器上添加 `data-multiselect` 以允许用户选择多个选项。每次点击切换项目。指示栏显示计数。

```html
<div class="options" data-multiselect>
  <!-- 相同的选项标记 — 用户可以选择/取消选择多个 -->
</div>
```

### 卡片（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup 内容 --></div>
    <div class="card-body">
      <h3>名称</h3>
      <p>描述</p>
    </div>
  </div>
</div>
```

### Mockup 容器

```html
<div class="mockup">
  <div class="mockup-header">预览：仪表盘布局</div>
  <div class="mockup-body"><!-- 你的 mockup HTML --></div>
</div>
```

### 分割视图（并排）

```html
<div class="split">
  <div class="mockup"><!-- 左侧 --></div>
  <div class="mockup"><!-- 右侧 --></div>
</div>
```

### 优点/缺点

```html
<div class="pros-cons">
  <div class="pros"><h4>优点</h4><ul><li>益处</li></ul></div>
  <div class="cons"><h4>缺点</h4><ul><li>不足</li></ul></div>
</div>
```

### Mock 元素（线框图构建块）

```html
<div class="mock-nav">Logo | 首页 | 关于 | 联系</div>
<div style="display: flex;">
  <div class="mock-sidebar">导航</div>
  <div class="mock-content">主要内容区域</div>
</div>
<button class="mock-button">操作按钮</button>
<input class="mock-input" placeholder="输入字段">
<div class="placeholder">占位区域</div>
```

### 排版和分区

- `h2` — 页面标题
- `h3` — 章节标题
- `.subtitle` — 标题下方的辅助文字
- `.section` — 带底部边距的内容块
- `.label` — 小型大写标签文字

## 浏览器事件格式

当用户在浏览器中点击选项时，他们的交互被记录到 `$STATE_DIR/events`（每行一个 JSON 对象）。当你推送新画面时，文件会自动清除。

```jsonl
{"type":"click","choice":"a","text":"选项 A - 简洁布局","timestamp":1706000101}
{"type":"click","choice":"c","text":"选项 C - 复杂网格","timestamp":1706000108}
{"type":"click","choice":"b","text":"选项 B - 混合布局","timestamp":1706000115}
```

完整的事件流展示用户的探索路径——他们可能在最终确定之前点击多个选项。最后一个 `choice` 事件通常是最终选择，但点击的模式可以揭示犹豫或值得追问的偏好。

如果 `$STATE_DIR/events` 不存在，说明用户没有与浏览器交互——只使用他们的终端文字。

## 设计技巧

- **根据问题调整保真度** — 布局问题用线框图，润色问题用高保真
- **在每个页面上解释问题** — "哪种布局感觉更专业？"而不仅仅是"选一个"
- **推进之前先迭代** — 如果反馈改变了当前画面，编写新版本
- **每个画面最多 2-4 个选项**
- **当内容重要时使用真实内容** — 对于摄影作品集，使用实际图片（Unsplash）。占位内容会掩盖设计问题。
- **保持 mockup 简洁** — 专注于布局和结构，而非像素级完美的设计

## 文件命名

- 使用语义化名称：`platform.html`、`visual-style.html`、`layout.html`
- 切勿重用文件名 — 每个画面必须是新文件
- 迭代版本：追加版本后缀，如 `layout-v2.html`、`layout-v3.html`
- 服务器按修改时间提供最新文件

## 清理

```bash
scripts/stop-server.sh $SESSION_DIR
```

如果会话使用了 `--project-dir`，mockup 文件保留在 `.superpowers/brainstorm/` 中以供后续参考。只有 `/tmp` 的会话在停止时会被删除。

## 参考

- 框架模板（CSS 参考）：`scripts/frame-template.html`
- 辅助脚本（客户端）：`scripts/helper.js`
