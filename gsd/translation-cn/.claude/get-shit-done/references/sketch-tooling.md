# 草图工具栏

在每个草图中包含一个小型浮动工具栏。它提供实用功能而不与设计竞争。

## 实现

一个固定在右下角的小 `<div>`，半透明，hover 时展开：

```html
<div id="sketch-tools" style="position:fixed;bottom:12px;right:12px;z-index:9999;font-family:system-ui;font-size:12px;background:rgba(0,0,0,0.7);color:white;padding:8px 12px;border-radius:8px;opacity:0.4;transition:opacity 0.2s;" onmouseenter="this.style.opacity='1'" onmouseleave="this.style.opacity='0.4'">
  <!-- 主题切换器 -->
  <!-- 视口按钮 -->
  <!-- 注释切换 -->
</div>
```

## 组件

### 主题切换器

一个在运行时切换主题 CSS 文件的下拉菜单：

```html
<select onchange="document.querySelector('link[href*=themes]').href='../themes/'+this.value+'.css'">
  <option value="default">默认</option>
</select>
```

### 视口预览

三个按钮将草图内容区域约束为标准宽度：

- 手机：375px
- 平板：768px
- 桌面：1280px（或全宽）

通过将草图内容包裹在容器中并调整其 `max-width` 来实现。

### 注释模式

一个切换开关，hover 时覆盖间距值、颜色 hex 码和字体大小。实现为读取计算样式并在 tooltip 中展示的 JS 片段。帮助理解视觉决策而无需打开开发者工具。

## 样式

工具栏应不显眼——小、暗、半透明。它绝不应在视觉上与草图竞争。独立于主题样式（硬编码暗色背景、白色文本）。
