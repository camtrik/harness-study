# 多变体 HTML 模式

每个草图在同一个 HTML 文件中产出 2-3 个变体。用户在它们之间切换以进行比较。

## 基于标签页的变体

标准方法：页面顶部的标签栏，每个标签显示不同的变体。

```html
<div id="variant-nav" style="position:fixed;top:0;left:0;right:0;z-index:9998;background:var(--color-surface, #fff);border-bottom:1px solid var(--color-border, #e5e5e5);padding:8px 16px;display:flex;gap:8px;font-family:system-ui;">
  <button class="variant-tab active" onclick="showVariant('a')">A: 侧边栏布局</button>
  <button class="variant-tab" onclick="showVariant('b')">B: 顶部导航</button>
  <button class="variant-tab" onclick="showVariant('c')">C: 浮动面板</button>
</div>

<div id="variant-a" class="variant active">
  <!-- 变体 A 内容 -->
</div>
<div id="variant-b" class="variant" style="display:none">
  <!-- 变体 B 内容 -->
</div>
<div id="variant-c" class="variant" style="display:none">
  <!-- 变体 C 内容 -->
</div>

<script>
function showVariant(id) {
  document.querySelectorAll('.variant').forEach(v => v.style.display = 'none');
  document.querySelectorAll('.variant-tab').forEach(t => t.classList.remove('active'));
  document.getElementById('variant-' + id).style.display = 'block';
  event.target.classList.add('active');
}
</script>
```

为 body 添加 `padding-top` 以适配固定的标签栏。

## 标记胜出者

在用户选择方向后，在胜出的标签上添加视觉指示器：

```html
<button class="variant-tab active">A: 侧边栏布局 ★ 已选择</button>
```

保持所有变体可见和可导航——胜出者被高亮，而非唯一选项。

## 并排（用于小型变体）

当比较小型元素（按钮样式、卡片布局、图标处理）时，将它们并排渲染并标注，而非使用标签页：

```html
<div style="display:grid;grid-template-columns:repeat(3,1fr);gap:24px;padding:24px;">
  <div>
    <h3>A: 圆角</h3>
    <!-- 变体内容 -->
  </div>
  <div>
    <h3>B: 锐利</h3>
    <!-- 变体内容 -->
  </div>
  <div>
    <h3>C: 药丸</h3>
    <!-- 变体内容 -->
  </div>
</div>
```

## 变体数量

- **第一轮（戏剧性的）：** 2-3 个显著不同的方法
- **细化轮：** 在选定方向内的 2-3 个微妙变体
- **绝不超过 4 个**——比这多会让人不知所措。如果有 5+ 个选项，在展示前收窄。

## 综合变体

当用户在变体间挑拣元素时，创建一个新的变体标签，描述性命名：

```html
<button class="variant-tab" onclick="showVariant('synth1')">综合：A 的布局 + C 的调色板</button>
```
