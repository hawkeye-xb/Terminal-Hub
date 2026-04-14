# DESIGN — 视觉设计与交互

## 色彩方案

3 种内置主题，通过 CSS 变量切换。

### Green（默认）

```css
html[data-theme="green"] {
  --bg:         #0d0d0d;
  --text:       #00ff00;
  --text-dim:   #00aa00;
  --accent:     #00ff00;
  --border:     #333333;
  --surface:    #1a1a1a;
  --glow:       rgba(0, 255, 0, 0.3);
}
```

对比度：#00ff00 on #0d0d0d = 15.4:1（超过 WCAG AAA）

### Amber

```css
html[data-theme="amber"] {
  --bg:         #1a1410;
  --text:       #ffaa22;
  --text-dim:   #cc8800;
  --accent:     #ffaa22;
  --border:     #3a2a1a;
  --surface:    #2a1e14;
  --glow:       rgba(255, 170, 34, 0.3);
}
```

### Cyber

```css
html[data-theme="cyber"] {
  --bg:         #0a0e27;
  --text:       #00d9ff;
  --text-dim:   #0088cc;
  --accent:     #00ffff;
  --border:     #1a2a4a;
  --surface:    #0f1535;
  --glow:       rgba(0, 217, 255, 0.3);
}
```

---

## 字体

```css
font-family: 'JetBrains Mono', 'Source Code Pro', 'Fira Code', monospace;
```

| 用途 | 大小 | 行高 |
|------|------|------|
| 正文 | 16px | 1.6 |
| h1 | 28px | 1.3 |
| h2 | 22px | 1.4 |
| h3 | 18px | 1.4 |
| 标签/日期 | 12px | 1.4 |
| CLI 输入 | 14px | 1.4 |

字体自托管，不依赖 Google Fonts。

---

## 间距

基于 8px 倍数：

```css
--sp-1: 4px;
--sp-2: 8px;
--sp-3: 16px;
--sp-4: 24px;
--sp-5: 32px;
--sp-6: 48px;
```

最大内容宽度：800px。

---

## 页面布局

### 首页

```
┌──────────────────────────────────────────┐
│  ~/                            [theme]   │ 48px  ← 路径导航 + 主题切换
├──────────────────────────────────────────┤
│                                          │
│  ┌─────────┐                             │
│  │ [头像]  │  名字                       │ 个人信息区
│  │ 方形    │  职业标签                   │ ~120px
│  │ 80x80   │  一句话简介                 │
│  └─────────┘  社交链接图标               │
│                                          │
│  ─── PROJECTS ─────────────────────────  │
│                                          │
│  ┌──────┐  ┌──────┐  ┌──────┐           │ 项目区
│  │ Proj │  │ Proj │  │ Proj │           │ ~150px
│  │  1   │  │  2   │  │  3   │           │
│  └──────┘  └──────┘  └──────┘           │
│                                          │
│  ─── RECENT ───────────────────────────  │
│                                          │
│  [日期] [类型] 标题                      │ 内容流区
│  [日期] [类型] 标题                      │ 动态高度
│  [日期] [类型] 标题                      │
│  ...                                     │
│                                          │
│  (底部留 60px 空间给 CLI)                │
│                                          │
├──────────────────────────────────────────┤
│  > /_ (浮动指令条)                       │ 50px fixed
└──────────────────────────────────────────┘
```

**没有侧边栏。** 单列居中布局，最大宽度 800px。简洁。

### 文章页 (single.html)

标准的 Hugo 文章页面。顶部有面包屑导航，底部有上一篇/下一篇链接。这是 GUI 模式下用户点击文章标题到达的页面。

### 项目页 (projects/single.html)

项目名、状态、版本、技术栈、描述、链接、相关文章。

### 列表页 (list.html)

分类/标签的文章列表。标准的 Hugo 列表页。

---

## 响应式

| 断点 | 布局变化 |
|------|--------|
| < 640px（手机） | 项目卡片改为纵向堆叠，头像缩小到 60px |
| 640-1024px（平板） | 项目卡片 2 列 |
| > 1024px（桌面） | 项目卡片 3 列 |

CLI 指令条在所有设备上始终底部固定。

手机端点击区域最小 44px。

---

## 组件

### 项目卡片

```css
.project-card {
  padding: var(--sp-3);
  border: 1px solid var(--border);
  background: var(--surface);
}
.project-card:hover {
  border-color: var(--accent);
  box-shadow: 0 0 12px var(--glow);
}
```

内容：项目名、状态标记（● Stable / ◐ Alpha / ○ Archive）、一行描述。

### 内容流条目

```css
.stream-item {
  padding: var(--sp-2) 0;
  border-bottom: 1px solid var(--border);
  display: flex;
  gap: var(--sp-2);
}
```

内容：`[日期] [类型标签] 标题`。类型标签不同颜色区分来源。

### 标签

```css
.tag {
  padding: 2px 8px;
  border: 1px solid var(--border);
  font-size: 12px;
  color: var(--text-dim);
}
.tag:hover {
  color: var(--bg);
  background: var(--accent);
}
```

### Header 路径导航

```css
.site-header {
  height: 48px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 var(--sp-3);
  border-bottom: 1px solid var(--border);
}
.path {
  font-size: 14px;
  color: var(--text-dim);
}
.path a {
  color: var(--text);
  text-decoration: none;
}
.path a:hover {
  color: var(--accent);
  text-decoration: underline;
}
```

路径示例：`~/` → `~/articles` → `~/articles/rust-guide`，每段可点击跳回。

### CLI 指令条

```css
.cli-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 50px;
  background: var(--bg);
  border-top: 1px solid var(--border);
  display: flex;
  align-items: center;
  padding: 0 var(--sp-3);
  z-index: 100;
}
.cli-bar::before {
  content: '> ';
  color: var(--accent);
  font-weight: bold;
}
.cli-bar input {
  flex: 1;
  background: none;
  border: none;
  color: var(--text);
  font: inherit;
  outline: none;
}
.cli-bar.active {
  box-shadow: inset 0 0 20px var(--glow);
}
```

### 指令选择菜单

```css
.cli-menu {
  position: fixed;
  bottom: 50px;
  left: 0;
  right: 0;
  max-height: 300px;
  background: var(--surface);
  border-top: 1px solid var(--border);
  overflow-y: auto;
}
.cli-menu-item {
  padding: var(--sp-2) var(--sp-3);
  display: flex;
  justify-content: space-between;
}
.cli-menu-item.selected {
  background: var(--accent);
  color: var(--bg);
}
```

---

## 动画

仅 3 个动画，全部使用 GPU 加速属性：

```css
/* 1. CLI 激活时的光晕呼吸 */
@keyframes glow-pulse {
  0%, 100% { box-shadow: inset 0 0 10px var(--glow); }
  50%      { box-shadow: inset 0 0 25px var(--glow); }
}

/* 2. 视图切换淡入 */
@keyframes fade-in {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* 3. 主题切换过渡 */
html { transition: background-color 0.3s, color 0.3s; }
```

不做打字机效果（太花哨）。不做页面加载动画（影响首屏速度）。

---

## 可访问性

- 所有主题的文字/背景对比度 > 7:1
- Tab 键可以遍历所有可交互元素
- CLI 指令条有 `role="search"` 和 `aria-label`
- 项目卡片用 `<article>` 语义标签
- 图片有 alt 文本
