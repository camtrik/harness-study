# 共享主题系统

所有草图共享一个 CSS 变量主题，使设计决策在草图间累积。

## 设置

在第一个草图上，创建 `.planning/sketches/themes/` 并设置默认主题：

```
.planning/sketches/
  themes/
    default.css         ← 所有草图链接到此
  001-dashboard-layout/
    index.html          ← 链接到 ../themes/default.css
```

## 主题文件结构

每个主题仅定义 CSS 自定义属性——无组件样式，无布局规则。只有视觉词汇表：

```css
:root {
  /* 颜色 */
  --color-bg: #fafafa;
  --color-surface: #ffffff;
  --color-border: #e5e5e5;
  --color-text: #1a1a1a;
  --color-text-muted: #6b6b6b;
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-accent: #f59e0b;
  --color-danger: #ef4444;
  --color-success: #22c55e;

  /* 排版 */
  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;

  /* 间距 */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;

  /* 形状 */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-full: 9999px;

  /* 阴影 */
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.07);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1);
}
```

调整默认主题以匹配在导入期间建立的情绪/方向。以上值是起点——更改颜色、字体、间距和形状以匹配约定的美学。

## 链接

每个草图都链接到主题：

```html
<link rel="stylesheet" href="../themes/default.css">
```

## 创建新主题

当草图揭示美学分叉时（"这应该感觉是临床的还是温暖的？"），将两者都创建为主题文件，而不是为此争论。用户可以切换并感受差异。

以描述性方式命名主题：`midnight.css`、`warm-minimal.css`、`brutalist.css`。

## 主题切换器

包含在每个草图中（草图工具栏的一部分）：

```html
<select id="theme-switcher" onchange="document.querySelector('link[href*=themes]').href='../themes/'+this.value+'.css'">
  <option value="default">默认</option>
</select>
```

通过列出可用主题文件动态填充选项，或硬编码已知主题。
