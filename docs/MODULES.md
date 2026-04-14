# MODULES — 4 个模块的详细规范

## 总览

| 模块 | 职责 | 产出文件 | 工作量 | 依赖 |
|------|------|---------|-------|------|
| M1 主题 UI | CSS + Hugo 模板 | layouts/、assets/css/ | 5 天 | 无 |
| M2 CLI 系统 | 指令解析、视图切换、键盘导航 | static/js/cli.js | 4 天 | M1 |
| M3 搜索 | Pagefind 集成 | static/js/search.js | 1 天 | M1 |
| M4 RSS 聚合 | 数据抓取脚本 + CI | scripts/、.github/ | 1 天 | 无 |

M1 和 M4 无依赖关系，可以并行。M2 和 M3 依赖 M1 的 HTML 结构。

**总计：~11 天（1 人），~6 天（2 人并行）。**

---

## M1. 主题 UI

### 职责

实现完整的 Hugo 主题模板和 CSS 样式。

### 交付物

```
layouts/
├── index.html                 # 首页
├── _default/baseof.html       # 基础模板
├── _default/single.html       # 文章详情
├── _default/list.html         # 列表页
├── projects/single.html       # 项目详情
├── projects/list.html         # 项目列表
├── partials/head.html         # <head>
├── partials/header.html       # 顶栏
├── partials/footer.html       # 底部
├── partials/cli.html          # CLI 指令条
├── partials/profile.html      # 个人信息区
├── partials/project-card.html # 项目卡片
├── partials/stream-item.html  # 内容流条目
└── 404.html                   # 404 页面

assets/css/
├── main.css                   # 主样式
└── themes.css                 # 3 种主题变量

static/fonts/
└── JetBrainsMono/             # 字体文件
```

### 需求细节

**首页 (index.html)**
- 个人信息区：左侧方形头像（80x80），右侧名字 + 职业 + 一句话 + 社交链接
- 项目区：3 列网格展示 `content/projects/` 的内容
- 内容流区：按时间倒序显示最近 10 条内容（文章 + 短文混排）
- 底部预留 60px 给浮动指令条

**文章详情 (single.html)**
- 标题、日期、标签、字数
- Markdown 渲染的正文内容
- 底部上一篇/下一篇导航

**项目详情 (projects/single.html)**
- 项目名、状态、版本
- 描述（Markdown）
- 技术栈标签
- 链接（Demo、GitHub、文档）
- 相关文章（通过 `relatedProject` 参数关联）

**CSS**
- 所有颜色通过 CSS 变量控制
- 响应式：640px 和 1024px 两个断点
- 无 CSS 框架依赖
- 最终产物 < 50KB（minified）

### 验收标准

```
- [ ] hugo server 能正常渲染 exampleSite
- [ ] 首页展示个人信息 + 3 个项目 + 最近内容
- [ ] 3 种主题可通过配置切换
- [ ] 手机端（320px）无水平滚动
- [ ] 文章详情页渲染正常（标题、代码块、图片）
- [ ] Lighthouse Performance > 90
```

---

## M2. CLI 系统

### 职责

实现斜杠指令系统：命令解析、视图切换、键盘导航、自动补全、历史记录。

### 交付物

```
static/js/cli.js              # 约 400-600 行
```

一个文件。不拆分模块（项目不大，拆了反而增加复杂度）。

### 需求细节

**状态机**

5 个状态：IDLE → MENU → LIST → DETAIL → SEARCH

状态转换规则见 VISION.md 的状态机图。

**指令解析**

支持的指令（复用真实终端语义）：

| 指令 | 来源 | 行为 | 状态转换 |
|------|------|------|---------|
| `cd <dir>` | Unix | 进入目录 | * → LIST（自动 ls） |
| `cd ..` | Unix | 回到上一级 | LIST → IDLE / DETAIL → LIST |
| `cd /` / `cd ~` | Unix | 回到首页 | * → IDLE |
| `ls` | Unix | 列出当前目录 | 刷新 LIST |
| `cat <N>` | Unix | 查看第 N 项 | LIST → DETAIL |
| `grep <kw>` | Unix | 搜索 | * → SEARCH |
| `tree` | Unix | 目录结构 | 显示全站树状图 |
| `pwd` | Unix | 显示当前路径 | 不切换（输出路径） |
| `clear` | Unix | 清屏回首页 | * → IDLE |
| `theme <name>` | 自定义 | 切换主题 | 不切换状态 |
| `help` / `man` | Unix | 显示帮助 | 显示帮助信息 |
| `history` | Bash | 显示历史 | 显示历史记录 |

路径导航：`cd articles` → `cd ../projects` → `cd /`，和真实终端一致。

**自动补全**

- 输入 `/` 后显示所有指令菜单
- 继续输入过滤菜单（`/a` → 只显示 `/articles`、`/about`）
- Tab 补全当前选中项
- ↑↓ 在菜单中导航

**历史记录**

- 保存最近 30 条指令到 LocalStorage
- ↑↓ 在输入框为空时浏览历史

**键盘导航**

| 键 | IDLE 状态 | MENU 状态 | LIST 状态 | DETAIL 状态 |
|----|----------|----------|----------|------------|
| `/` | 激活 CLI | — | — | — |
| Esc | — | 关闭菜单 | 回首页 | 回列表 |
| ↑ | — | 上一项 | 上一项 | 向上滚动 |
| ↓ | — | 下一项 | 下一项 | 向下滚动 |
| ← | — | — | — | 上一篇 |
| → | — | — | — | 下一篇 |
| Enter | — | 执行选中 | 打开选中 | — |
| Tab | — | 补全 | — | — |

**视图渲染**

CLI 模式的内容渲染在 `#cli-view` 容器内。数据来源是 Hugo 构建时注入到 `window.__TERMINAL_DATA__` 的 JSON。

列表视图每行格式：`[N] 标题  日期 · 标签`

详情视图：两种实现方式
1. **简单方式**：`window.location.href = item.url`（直接跳转到 Hugo 页面）
2. **高级方式**：`fetch(item.url)` 获取 HTML 并提取 `<article>` 内容，在 `#cli-view` 中展示

MVP 用简单方式。高级方式作为可选增强。

### 验收标准

```
- [ ] 按 / 激活 CLI，显示指令菜单
- [ ] cd articles / cd projects / cd moments 进入对应列表
- [ ] cd .. 回到上一级，cd / 回到首页
- [ ] cat N 打开第 N 项内容
- [ ] grep <keyword> 搜索正常
- [ ] Header 路径实时更新，每段可点击
- [ ] Tab 自动补全正常
- [ ] ↑↓ 菜单和列表导航正常
- [ ] Esc 等同于 cd ..
- [ ] 历史记录保存到 LocalStorage
- [ ] 指令执行响应 < 100ms
- [ ] JS 文件 < 30KB（minified）
- [ ] 无内存泄漏
```

---

## M3. 搜索

### 职责

集成 Pagefind，实现 `/search <keyword>` 指令。

### 交付物

```
static/js/search.js            # 约 50-80 行
```

### 需求细节

**构建时生成索引**

在 Hugo 构建完成后运行 `npx pagefind --site public`，生成搜索索引到 `public/_pagefind/`。

**运行时搜索**

```javascript
let pagefind = null;

export async function initSearch() {
  pagefind = await import('/_pagefind/pagefind.js');
  await pagefind.init();
}

export async function search(query) {
  if (!pagefind) await initSearch();
  const results = await pagefind.search(query);
  const items = await Promise.all(
    results.results.slice(0, 20).map(r => r.data())
  );
  return items.map(item => ({
    title: item.meta?.title || 'Untitled',
    url: item.url,
    excerpt: item.excerpt,
  }));
}
```

**与 CLI 集成**

在 cli.js 中：
```javascript
import { search } from './search.js';

commands.search = async (keyword) => {
  const results = await search(keyword);
  currentList = results;
  renderSearchResults(results, keyword);
  state = State.SEARCH;
};
```

搜索结果的展示格式和文章列表一致，支持 `view N` 打开。

### 验收标准

```
- [ ] /search <keyword> 返回相关结果
- [ ] 搜索响应 < 500ms
- [ ] 结果列表支持 view N
- [ ] 无结果时显示"未找到"
- [ ] Pagefind 索引在构建时自动生成
```

---

## M4. RSS 聚合（可选）

### 职责

从 RSSHub 拉取你在其他平台的内容，生成 `data/feeds.json`，在首页"RECENT"区域中展示。

**这个模块是可选的。** 不启用时，首页只展示 `content/` 目录中的本地内容。

### 交付物

```
scripts/sync-feeds.js          # 约 60 行
.github/workflows/sync.yml     # 约 25 行
package.json                   # rss-parser 依赖
```

### 需求细节

**脚本逻辑**

1. 读取 RSS 源列表（硬编码在脚本中或从配置读取）
2. 并发抓取所有源
3. 合并、排序、去重
4. 写入 `data/feeds.json`
5. 单个源失败不影响其他源

**Hugo 模板中使用**

```go
{{ range .Site.Data.feeds }}
  {{ partial "stream-item.html" . }}
{{ end }}
```

**GitHub Actions**

每 6 小时运行一次，自动 commit 并 push 更新后的 `data/feeds.json`。

### 验收标准

```
- [ ] 脚本运行成功，生成 data/feeds.json
- [ ] 单个源失败时其他源正常
- [ ] GitHub Actions 定时运行正常
- [ ] 首页能展示聚合内容
```

---

## 模块间接口

```
M1 (主题 UI)
  ├─ 提供 HTML 结构：#content, #cli-bar, #cli-menu, #cli-view
  ├─ 提供 CSS 类名：.cli-bar, .cli-menu, .list-item, ...
  └─ 在首页注入 window.__TERMINAL_DATA__

M2 (CLI)
  ├─ 读取 window.__TERMINAL_DATA__
  ├─ 操作 #cli-bar, #cli-menu, #cli-view 的 DOM
  ├─ 调用 M3 的 search() 函数
  └─ 切换 #content 和 #cli-view 的显示/隐藏

M3 (搜索)
  ├─ 暴露 search(query) 函数
  └─ 依赖 Pagefind 在构建时生成的索引

M4 (RSS 聚合)
  └─ 输出 data/feeds.json，供 M1 的 Hugo 模板读取
```

没有循环依赖。M1 是基础，M2 是核心交互，M3 是增强，M4 是可选。
