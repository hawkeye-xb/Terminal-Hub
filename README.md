# Terminal Hub — Hugo 终端风格个人主题

一个终端风格的 Hugo 主题。展示你的作品、文章和想法。

## 文档

| 文档 | 内容 |
|------|------|
| [VISION.md](docs/VISION.md) | 做什么、为什么、核心交互 |
| [DESIGN.md](docs/DESIGN.md) | 视觉设计、布局、组件 |
| [IMPLEMENTATION.md](docs/IMPLEMENTATION.md) | 目录结构、代码示例、开发计划 |
| [MODULES.md](docs/MODULES.md) | 4 个模块的详细 spec |

## 技术栈

- Hugo（静态生成）
- Vanilla JavaScript（交互，< 30KB）
- 纯 CSS（样式，< 50KB）
- Pagefind（全文搜索）
- Cloudflare Pages（部署）

## 快速开始

```bash
# 用这个主题创建站点
hugo new site my-site
cd my-site
git submodule add https://github.com/user/terminal-hub-theme themes/terminal-hub
echo 'theme = "terminal-hub"' >> hugo.toml

# 写内容
hugo new posts/hello-world.md
hugo new projects/my-app.md

# 本地预览
hugo server
```

## 配置

```toml
# hugo.toml
baseURL = "https://yoursite.com"
theme = "terminal-hub"

[params]
author = "Your Name"
bio = "Full-stack Developer"
avatar = "images/avatar.jpg"
colorScheme = "green" # green / amber / cyber

[params.social]
github = "https://github.com/you"
email = "you@example.com"
```

## 许可证

MIT
