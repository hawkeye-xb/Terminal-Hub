# IMPLEMENTATION — 开发指南

## 目录结构

这是一个 Hugo 主题，不是一个 Hugo 站点。

```
terminal-hub-theme/
├── theme.toml                    # Hugo 主题元数据
├── LICENSE
├── README.md
│
├── layouts/
│   ├── index.html                # 首页模板
│   ├── _default/
│   │   ├── baseof.html           # 基础模板（<html><head><body>）
│   │   ├── single.html           # 文章/短文详情页
│   │   └── list.html             # 分类/标签列表页
│   ├── projects/
│   │   ├── single.html           # 项目详情页
│   │   └── list.html             # 项目列表页
│   ├── partials/
│   │   ├── head.html             # <head> 标签内容
│   │   ├── header.html           # 站点顶部
│   │   ├── footer.html           # 站点底部
│   │   ├── cli.html              # CLI 指令条 + 菜单
│   │   ├── profile.html          # 个人信息区（左图右介）
│   │   ├── project-card.html     # 项目卡片
│   │   └── stream-item.html      # 内容流条目
│   └── 404.html
│
├── assets/
│   └── css/
│       ├── main.css              # 主样式（含变量、布局、组件）
│       └── themes.css            # 3 种主题的 CSS 变量
│
├── static/
│   ├── js/
│   │   ├── cli.js                # CLI 核心：状态机 + 命令解析 + UI
│   │   └── search.js             # Pagefind 集成
│   └── fonts/
│       └── JetBrainsMono/        # 自托管字体文件
│
└── exampleSite/                  # 示例站点（供演示和测试）
    ├── hugo.toml
    ├── content/
    │   ├── posts/
    │   │   ├── _index.md
    │   │   ├── rust-performance.md
    │   │   └── vue3-guide.md
    │   ├── projects/
    │   │   ├── _index.md
    │   │   └── xisper.md
    │   └── moments/
    │       ├── _index.md
    │       └── 2026-04-13-idea.md
    ├── data/
    │   └── feeds.json            # 可选：RSS 聚合数据
    └── static/
        └── images/
            └── avatar.jpg
```

**约 30 个文件。** CSS < 50KB，JS < 30KB，字体 ~100KB。

---

## 核心模板

### baseof.html

```html
<!DOCTYPE html>
<html lang="{{ .Site.Language }}" data-theme="{{ .Site.Params.colorScheme | default "green" }}">
<head>
  {{ partial "head.html" . }}
</head>
<body>
  <header class="site-header">
    <span id="path-display" class="path">~/</span>
    <button id="theme-toggle" class="theme-btn" aria-label="切换主题">[theme]</button>
  </header>
  <main id="content" class="container">
    {{ block "main" . }}{{ end }}
  </main>
  <div id="cli-view" class="container" hidden></div>
  {{ partial "cli.html" . }}
</body>
</html>
```

Header 就两个元素：左边的路径（`~/articles/rust-guide`），右边的主题切换按钮。路径每一段可点击。

### index.html（首页）

```html
{{ define "main" }}
  <!-- 个人信息 -->
  {{ partial "profile.html" . }}

  <!-- 项目矩阵 -->
  <section class="projects">
    <h2>PROJECTS</h2>
    <div class="project-grid">
      {{ range (where .Site.RegularPages "Section" "projects") }}
        {{ partial "project-card.html" . }}
      {{ end }}
    </div>
  </section>

  <!-- 最近内容 -->
  <section class="recent">
    <h2>RECENT</h2>
    {{ range (first 10 .Site.RegularPages) }}
      {{ partial "stream-item.html" . }}
    {{ end }}
  </section>
{{ end }}
```

### cli.html（指令条）

```html
<div class="cli-bar" id="cli-bar">
  <input type="text"
         id="cli-input"
         placeholder="按 / 输入指令"
         autocomplete="off"
         role="search"
         aria-label="命令行输入">
</div>
<div class="cli-menu" id="cli-menu" hidden></div>
<div class="cli-view" id="cli-view" hidden></div>

<script src="{{ "js/cli.js" | relURL }}"></script>
<script src="{{ "js/search.js" | relURL }}"></script>
```

`#cli-view` 是 CLI 模式下替换内容区的容器。当 CLI 激活并执行命令后，`#content`（Hugo 渲染的内容）被隐藏，`#cli-view` 显示。`/exit` 时反转。

---

## CLI 核心逻辑 (cli.js)

### 状态机

```javascript
const State = {
  IDLE: 'idle',           // 首页，CLI 未激活
  MENU: 'menu',           // CLI 激活，显示指令菜单
  LIST: 'list',           // 显示文章/项目/短文列表
  DETAIL: 'detail',       // 阅读文章/项目详情
  SEARCH: 'search',       // 搜索结果
};
```

### 数据来源

CLI 的内容数据从哪来？**从 Hugo 在构建时生成的 JSON。**

在 `layouts/index.html` 底部注入：

```html
<script>
  window.__TERMINAL_DATA__ = {
    posts: [
      {{ range (where .Site.RegularPages "Section" "posts") }}
      {
        title: {{ .Title | jsonify }},
        url: {{ .RelPermalink | jsonify }},
        date: {{ .Date.Format "2006-01-02" | jsonify }},
        summary: {{ .Summary | plainify | truncate 100 | jsonify }},
        tags: {{ .Params.tags | jsonify }},
        wordCount: {{ .WordCount }},
      },
      {{ end }}
    ],
    projects: [
      {{ range (where .Site.RegularPages "Section" "projects") }}
      {
        title: {{ .Title | jsonify }},
        url: {{ .RelPermalink | jsonify }},
        status: {{ .Params.status | default "stable" | jsonify }},
        description: {{ .Params.description | jsonify }},
        tech: {{ .Params.tech | jsonify }},
      },
      {{ end }}
    ],
    moments: [
      {{ range (where .Site.RegularPages "Section" "moments") }}
      {
        title: {{ .Title | jsonify }},
        url: {{ .RelPermalink | jsonify }},
        date: {{ .Date.Format "2006-01-02" | jsonify }},
        summary: {{ .Summary | plainify | truncate 100 | jsonify }},
      },
      {{ end }}
    ],
  };
</script>
```

这样 CLI 有了所有内容的索引，可以在客户端渲染列表，不需要额外请求。

### 路径状态

```javascript
// 当前路径，映射到网站目录结构
let currentPath = '/';  // '/' = 首页, '/articles' = 文章列表, '/articles/slug' = 文章详情

// 路径 → Hugo section 映射
const sections = {
  'articles': 'posts',
  'projects': 'projects',
  'moments': 'moments',
};
```

### 命令执行

```javascript
const commands = {
  cd(target)    { navigate(target); },
  ls()          { listCurrentDir(); },
  cat(n)        { openItem(parseInt(n, 10)); },
  grep(kw)      { performSearch(kw); },
  tree()        { renderTree(); },
  pwd()         { showMessage(currentPath); },
  clear()       { navigate('/'); },
  theme(name)   { setTheme(name); },
  help()        { renderHelp(); },
  history()     { renderHistory(); },
};

function executeCommand(input) {
  const parts = input.trim().split(/\s+/);
  const cmd = parts[0];
  const args = parts.slice(1);

  if (commands[cmd]) {
    commands[cmd](...args);
    addToHistory(input);
  } else {
    showError(`command not found: ${cmd}`);
  }
}

// cd 导航逻辑
function navigate(target) {
  if (!target || target === '/' || target === '~') {
    currentPath = '/';
    exitCliView();           // 回到首页
  } else if (target === '..') {
    const parts = currentPath.split('/').filter(Boolean);
    parts.pop();
    currentPath = '/' + parts.join('/');
    if (currentPath === '/') exitCliView();
    else listCurrentDir();   // 回到上一级并 ls
  } else if (target.startsWith('/') || target.startsWith('~')) {
    currentPath = target.replace('~', '');
    listCurrentDir();
  } else {
    // 相对路径
    currentPath = currentPath === '/' 
      ? '/' + target 
      : currentPath + '/' + target;
    listCurrentDir();
  }
  updateHeaderPath();        // 更新 Header 中的路径显示
}
```

Header 路径更新：

```javascript
function updateHeaderPath() {
  const header = document.getElementById('path-display');
  // 把路径的每一段变成可点击的链接
  const parts = currentPath.split('/').filter(Boolean);
  let html = '<a href="#" onclick="navigate(\'/\')">~</a>';
  let accumulated = '';
  for (const part of parts) {
    accumulated += '/' + part;
    const p = accumulated;
    html += '/<a href="#" onclick="navigate(\'' + p + '\')">' + part + '</a>';
  }
  header.innerHTML = html;
}
```
```

### 列表渲染

```javascript
let currentList = [];

function listCurrentDir() {
  // 从路径推断 section
  const dir = currentPath.split('/').filter(Boolean)[0];
  const section = sections[dir];
  if (!section) {
    showError(`no such directory: ${dir}`);
    return;
  }
  
  const data = window.__TERMINAL_DATA__[section];
  currentList = data;

  const html = data.map((item, i) => `
    <div class="list-item" data-index="${i + 1}">
      <span class="list-num">[${i + 1}]</span>
      <div>
        <div class="list-title">${item.title}</div>
        <div class="list-meta">${item.date || ''} · ${item.tags?.join(' ') || ''}</div>
      </div>
    </div>
  `).join('');

  showCliView(`
    <div class="cli-list">
      <div class="cli-prompt">$ cd ${dir} && ls</div>
      <h2>${dir.toUpperCase()} (${data.length})</h2>
      ${html}
      <div class="list-hint">↑↓ 导航  Enter/cat N 查看  cd .. 返回</div>
    </div>
  `);

  state = State.LIST;
}

function openItem(n) {
  if (n < 1 || n > currentList.length) {
    showError(`cat: ${n}: No such file`);
    return;
  }
  const item = currentList[n - 1];
  const slug = item.url.split('/').filter(Boolean).pop();
  currentPath = currentPath + '/' + slug;
  updateHeaderPath();
  
  // 简单方式：直接跳转
  window.location.href = item.url;
}
```
```

### view N 的高级实现（可选）

如果想在 CLI 视图内阅读而不跳转页面，可以用 fetch 获取文章页并提取内容：

```javascript
async function openItemInline(n) {
  const item = currentList[n - 1];
  const resp = await fetch(item.url);
  const html = await resp.text();
  const doc = new DOMParser().parseFromString(html, 'text/html');
  const articleContent = doc.querySelector('article')?.innerHTML || doc.querySelector('main')?.innerHTML;

  showCliView(`
    <div class="cli-article">
      <h1>${item.title}</h1>
      <div class="article-meta">${item.date} · ${item.tags?.join(' ') || ''}</div>
      <div class="article-body">${articleContent}</div>
      <div class="list-hint">↑↓ 滚动  ←→ 上一篇/下一篇  /exit 返回列表</div>
    </div>
  `);

  state = State.DETAIL;
}
```

---

## 键盘事件

```javascript
document.addEventListener('keydown', (e) => {
  // / 键：激活 CLI（和 vim 的搜索键一致）
  if (e.key === '/' && state === State.IDLE) {
    e.preventDefault();
    activateCli();
    return;
  }

  // Esc：退出当前状态（等同于 cd ..）
  if (e.key === 'Escape') {
    if (state === State.MENU) deactivateCli();
    else navigate('..');
    return;
  }

  // ↑↓：在列表或菜单中导航 / 在文章中滚动
  if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
    if (state === State.MENU) navigateMenu(e.key);
    if (state === State.LIST) navigateList(e.key);
    if (state === State.DETAIL) scrollArticle(e.key);
    return;
  }

  // ←→：在文章详情中翻页（上一篇 / 下一篇）
  if (state === State.DETAIL) {
    if (e.key === 'ArrowLeft') openItem(currentIndex - 1);
    if (e.key === 'ArrowRight') openItem(currentIndex + 1);
  }

  // Enter：在列表中打开选中项（等同于 cat N）
  if (e.key === 'Enter' && state === State.LIST) {
    openItem(selectedIndex);
  }
});
```

---

## RSS 聚合脚本（可选功能）

`scripts/sync-feeds.js`：

```javascript
import Parser from 'rss-parser';
import { writeFileSync } from 'fs';

const parser = new Parser();

const sources = [
  { name: 'jike', url: 'https://rsshub.app/jike/user/YOUR_ID' },
  { name: 'xiaohongshu', url: 'https://rsshub.app/xiaohongshu/user/YOUR_ID' },
];

async function sync() {
  const items = [];
  for (const src of sources) {
    try {
      const feed = await parser.parseURL(src.url);
      for (const entry of feed.items) {
        items.push({
          source: src.name,
          title: entry.title,
          link: entry.link,
          date: entry.pubDate,
          summary: entry.contentSnippet?.slice(0, 200),
        });
      }
    } catch (e) {
      console.error(`Failed to fetch ${src.name}: ${e.message}`);
    }
  }
  items.sort((a, b) => new Date(b.date) - new Date(a.date));
  writeFileSync('data/feeds.json', JSON.stringify(items.slice(0, 50), null, 2));
  console.log(`Synced ${items.length} items`);
}

sync();
```

GitHub Actions 配置（`.github/workflows/sync.yml`）：

```yaml
name: Sync Feeds
on:
  schedule:
    - cron: '0 */6 * * *'
  workflow_dispatch:
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: node scripts/sync-feeds.js
      - run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git add data/feeds.json
          git diff --cached --quiet || git commit -m "sync feeds"
          git push
```

---

## 开发计划

### Week 1：主题骨架 + CSS

- [ ] 创建 Hugo 主题目录结构
- [ ] baseof.html、head.html、header.html、footer.html
- [ ] main.css：变量、布局、组件样式
- [ ] themes.css：3 种主题
- [ ] index.html 首页模板
- [ ] 响应式适配
- [ ] exampleSite 基本内容

验证：`hugo server` 能正常渲染首页，3 种主题可切换。

### Week 2：CLI 系统

- [ ] cli.html 模板
- [ ] cli.js：状态机、命令解析、菜单 UI
- [ ] 列表渲染（/articles、/projects、/moments）
- [ ] view N 功能
- [ ] 键盘导航（↑↓←→、Tab、Esc）
- [ ] 自动补全
- [ ] 历史记录（LocalStorage）

验证：所有指令可用，键盘导航流畅。

### Week 3：搜索 + 优化 + 发布

- [ ] Pagefind 集成
- [ ] /search 指令接入 Pagefind
- [ ] SEO（meta tags、sitemap、Open Graph）
- [ ] 性能优化（CSS/JS 压缩、字体子集化）
- [ ] README 完善
- [ ] exampleSite 完整内容
- [ ] Lighthouse 测试
- [ ] 发布到 GitHub

验证：Lighthouse > 90，别人能按 README 跑起来。

---

## 本地开发

```bash
cd terminal-hub-theme/exampleSite
hugo server --themesDir ../..
# 打开 http://localhost:1313
```

修改主题文件后 Hugo 会自动热重载。

---

## 部署

使用者（不是主题开发者）的部署流程：

```bash
# 创建站点
hugo new site my-site && cd my-site
git init
git submodule add https://github.com/user/terminal-hub-theme themes/terminal-hub

# 配置
echo 'theme = "terminal-hub"' >> hugo.toml

# 写内容
hugo new posts/hello.md
hugo new projects/my-app.md

# 推到 GitHub，连接 Cloudflare Pages
# 构建命令：hugo
# 发布目录：public
```
