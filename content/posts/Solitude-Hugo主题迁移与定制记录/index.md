---
title: Solitude Hugo 主题迁移与定制记录
date: 2026-10-31T00:00:00+08:00
draft: false
featured: false
summary: 记录本站切换到 Solitude 后的配置、文章样式、封面取色、Waline 评论、侧栏和 GitHub Pages 发布方式，方便后续维护。
categories:
  - Hugo
tags:
  - Solitude
  - Hugo
  - Waline
  - GitHub Pages
featured_image: Pasted image 20261031214724.png
featured_image_preview: Pasted image 20261031214724.png
---
这篇文章记录本站从旧主题切换到 Solitude 后的配置和定制点。以后升级主题或调整样式时，可以先从这里找到配置入口和相关文件。

## 主题与构建环境

本站使用 `themes/solitude`。默认配置 `config/_default/hugo.toml` 中的 `theme` 已设置为 `solitude`，旧的 FixIt 和 Arberia 主题已清理。Solitude 的站点设置集中在 `config/pehtheme/hugo.toml`，包括中文语言、站点名称、Logo、首页横幅、代码高亮、评论和侧栏等参数。

GitHub Pages 工作流 `.github/workflows/hugo.yaml` 使用 `--environment pehtheme` 构建，所以本地预览也要选择相同环境：

```bash
cd /mnt/truenas-nas/hugo
hugo server --environment pehtheme
```

也可以用环境变量形式：

```bash
HUGO_ENVIRONMENT=pehtheme hugo server
```

直接执行 `hugo server` 会默认使用 `development` 环境，只读 `_default` 和 `development` 配置，不会自动加载 `config/pehtheme`。因此它可能显示 Solitude 的默认设置，而不是线上使用的完整配置。

当前 GitHub Actions 固定使用 Hugo `0.167.0`，构建命令会压缩输出并使用 Pages 提供的站点地址。Solitude 的项目资源是 CSS 和 TypeScript；主题没有 Sass/SCSS 源文件，也没有 Dart Sass 编译步骤，工作流不再启用 Hugo Extended。

## 代码块与目录

代码块样式、复制按钮、行号和最大高度在 `config/pehtheme/hugo.toml` 的 `[params.solitude.highlight]` 配置：

```toml
[params.solitude.highlight]
  enable = true
  copy = true
  line_numbers = true
  max_height = 360
  [params.solitude.highlight.themes]
    light = "github-light"
    dark = "github-dark"
```

文章目录使用 Hugo 的 `markup.tableOfContents` 设置，标题层级为二级至五级。文章中的 `##` 二级标题会显示 Solitude 自带的顶部虚线；这是主题样式，不是文章内容里的分隔符。单独的 Markdown `---` 也会渲染成水平线，但 front matter 开头和结尾的 `---` 不会显示在正文中。

## 首页、文章头图与导航

首页横幅名称和介绍来自 `config/pehtheme/hugo.toml` 的 `params.solitude.hometop.banner`。首页移动端轮播做了三项调整：

- 在轮播左右两侧加入上一张、下一张按钮，按钮只在手机宽度显示，桌面布局保持原样。按钮 HTML 在 `themes/solitude/layouts/_partials/home-top.html`，尺寸、位置和圆形半透明样式在 `themes/solitude/assets/css/solitude/pages/home.css`。
- 修复手机触摸后轮播卡片无法打开文章的问题。现在轻点卡片会进入文章，横向滑动只切换轮播；指示点和左右按钮也会同步当前卡片。事件逻辑在 `themes/solitude/assets/ts/main.ts`。
- 手机轮播采用约 `1.9:1` 的横图比例，并使用横向滚动吸附；桌面轮播不套用手机的滚动容器样式。

首页文章卡片图片也调整为较宽的横图比例：手机 CSS 使用 `1000 / 420`，图片以 `object-fit: cover` 填充。桌面卡片图片维持主题的固定高度，宽屏约 225px，窄桌面约 220px。相关规则在 `themes/solitude/assets/css/solitude/components/post-feed.css`。

对应 CSS 如下，图片由容器裁切而不是强行拉伸：

```css
@media screen and (max-width: 768px) {
  .recent-post-item .post_cover {
    width: 100%;
    height: auto;
    aspect-ratio: 1000 / 420;
  }
}

.recent-post-item .post_cover img.post_bg {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

手机菜单保留 Solitude 的布局，只调整了菜单项选择器的范围，避免父级菜单的悬停样式错误套用到子项。页面模板也包含 About 和归档页：About 页面在 `content/about/_index.md`，资料内容在 `data/about.yaml`；归档页由 `content/archives/_index.md` 指定类型。

### 添加轮播按钮

在 `themes/solitude/layouts/_partials/home-top.html` 的 `.home-center-banner` 外包一层 `.home-center-banner-wrap`，并在轮播容器后添加两个按钮。按钮用 `data-home-center-nav` 标记方向，供 TypeScript 统一处理：

```html
<div class="home-center-banner-wrap">
  <div class="home-center-banner">…轮播卡片…</div>
  <button class="home-center-nav home-center-nav-prev" type="button"
    data-home-center-nav="-1" aria-label="上一张推荐">‹</button>
  <button class="home-center-nav home-center-nav-next" type="button"
    data-home-center-nav="1" aria-label="下一张推荐">›</button>
</div>
```

桌面先隐藏按钮，手机端才显示圆形按钮；`display: contents` 让外层 wrapper 在桌面端不改变原本布局：

```css
.home-center-banner-wrap { display: contents; }
.home-center-nav { display: none; }

@media (max-width: 768px) {
  .home-center-banner-wrap { display: block; position: relative; }
  .home-center-nav {
    position: absolute;
    z-index: 4;
    top: 50%;
    display: grid;
    width: 2.25rem;
    height: 2.25rem;
    place-items: center;
    border: 1px solid rgb(255 255 255 / 35%);
    border-radius: 9999px;
    background: rgb(0 0 0 / 42%);
    color: #fff;
    transform: translateY(-50%);
  }
  .home-center-nav-prev { left: .75rem; }
  .home-center-nav-next { right: .75rem; }
}
```

按钮监听器加在 `themes/solitude/assets/ts/main.ts` 的 `initHomeCenter()` 中。它根据当前序号和方向循环选择卡片，然后滚动到对应卡片：

```ts
container.querySelectorAll<HTMLButtonElement>("[data-home-center-nav]")
  .forEach((button) => {
    button.addEventListener("click", (event) => {
      event.preventDefault();
      event.stopPropagation();
      const offset = Number(button.dataset.homeCenterNav || 0);
      const nextIndex = (activeIndex + offset + banners.length) % banners.length;
      select(nextIndex);
      banner.scrollTo({ left: banner.clientWidth * nextIndex, behavior: "smooth" });
    });
  });
```

手机横幅设为 `aspect-ratio: 1.9 / 1`，配合 `scroll-snap-type: x mandatory` 实现滑动吸附。轮播卡片的点击修复需要区分轻点和滑动：在 `pointerdown` 记录触摸坐标，移动超过 10px 就记为滑动；只有非滑动的触摸才执行 `navigate(item.dataset.link, event)`。不要在手机端一律 `preventDefault()`，否则卡片会无法进入文章。

触摸识别的关键逻辑如下，仍在 `initHomeCenter()`：

```ts
let pointerType = "";
let touchStartX = 0;
let touchStartY = 0;
let touchMoved = false;

banner.addEventListener("pointerdown", (event) => {
  pointerType = event.pointerType;
  touchStartX = event.clientX;
  touchStartY = event.clientY;
  touchMoved = false;
});

banner.addEventListener("pointermove", (event) => {
  if (pointerType === "touch" &&
      (Math.abs(event.clientX - touchStartX) > 10 ||
       Math.abs(event.clientY - touchStartY) > 10)) {
    touchMoved = true;
  }
});

// 卡片 click 中：touchMoved 时只结束滑动；轻点时才 navigate。
```

文章头图的样式经过几轮手机和宽屏调整，最终保留 Solitude 的背景渐变/遮罩效果，桌面宽屏头图布局回到主题默认样式。手机端头图高度改为 `max(13rem, 100vw / 1.9)`，标题仍叠放在图片中间，不移到图片下方；标题允许换行，并加白色字体描边和阴影，避免在浅色封面上看不清或长标题挤出画面。原创、分类、阅读量、日期、更新时间和阅读时长等信息按钮加半透明深色底、浅色描边和模糊效果。文章顶部的位置标签隐藏。这些自定义规则在 `assets/css/custom.css`；Solitude 的文章头图布局规则在 `themes/solitude/assets/css/solitude/pages/post.css`。

主要自定义规则可以直接加到 `assets/css/custom.css`：

```css
.post #post-info #post-meta .post-meta-position { display: none; }

.post #post-info .post-title {
  color: #fff;
  -webkit-text-stroke: 0.55px rgba(18, 20, 26, 0.82);
  paint-order: stroke fill;
  text-shadow: 0 1px 3px rgba(0, 0, 0, 0.62);
}

.post #post-info #post-firstinfo .meta-firstline .post-meta-original,
.post #post-info #post-firstinfo .meta-firstline span.post-meta-categories,
.post #post-info #post-meta .meta-secondline > span:not(.post-meta-separator),
.post #post-info #post-meta .meta-secondline > a.post-meta-pv {
  border: 1px solid rgba(255, 255, 255, 0.5);
  background: rgba(25, 27, 34, 0.34);
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(6px);
}

.post #post-info #post-firstinfo .meta-firstline .post-meta-original,
.post #post-info #post-firstinfo .meta-firstline span.post-meta-categories a,
.post #post-info #post-meta,
.post #post-info #post-meta a {
  color: #fff;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.55);
}

@media (max-width: 768px) {
  .post #page-header.post-bg {
    height: max(13rem, calc(100vw / 1.9));
    min-height: 0;
  }
  .post .post-cover-aside { height: 100%; }
  .post #post-info {
    padding: 3.4rem 0.75rem 1.2rem;
    background-image: linear-gradient(
      to bottom,
      rgba(20, 22, 28, 0.08) 0%,
      rgba(20, 22, 28, 0.18) 38%,
      var(--efu-main) 100%
    );
  }
  .post #post-info::after { display: none; }
}
```

标题描边粗细、阴影深浅可以按图片对比度微调；手机图高由 `1.9` 比例和 `13rem` 最小高度控制。不要覆盖整个桌面头图的高度规则，桌面布局已恢复 Solitude 默认值。

桌面端向下滚动后，顶部固定导航里的页面标题显示“返回顶部”，避免把回顶操作误认成站点名称。相关规则也在 `assets/css/custom.css`。

## 根据封面图自动配色

`config/pehtheme/hugo.toml` 启用了 Solitude 的本地封面取色：

```toml
[params.solitude.post.covercolor]
  enable = true
  mode = "local"
```

主题从文章封面提取颜色，用于文章链接、按钮、目录高亮和悬停效果。Solitude 原本只把提取色写入 `--efu-main`；部分夜间模式组件直接读取 `--efu-theme`，所以会继续显示默认黄色。

本站在 `themes/solitude/assets/ts/covercolor/shared.ts` 中补上了同步：取色时同时更新 `--efu-theme` 及透明度变量；没有可用封面色时则清除覆盖值并恢复主题默认色。这样日间和夜间模式里的文章链接、文章底部标签、“随便逛逛”按钮、推荐文章悬停效果、目录当前项和侧栏标签悬停都能跟随封面颜色。

## Waline 评论与侧栏

Waline 服务端地址和深色模式选择器配置在 `config/pehtheme/hugo.toml`：

```toml
[params.solitude.waline]
  serverURL = "https://hugo-waline-puce.vercel.app"
  pageview = false
  dark = 'html[data-theme="dark"]'
```

评论功能使用 Waline，并启用评论数量和侧栏评论：

```toml
[params.solitude.comment]
  use = "waline"
  count = true
  sidebar = true

[params.solitude.comment.newest_comment]
  enable = true
  limit = 5
```

页面侧栏和控制台的最新评论组件也已打开。文章页侧栏按配置显示最新文章、最新评论和站点信息；首页及普通页面使用各自的侧栏顺序。Waline 评论数据读取、最新评论汇总、评论数和首页文章卡片评论者头像由 `themes/solitude/assets/ts/comments.ts` 处理，侧栏模板在 `themes/solitude/layouts/_partials/sidebar.html`。

`assets/css/custom.css` 把 Waline 的背景、文字、边框和按钮变量映射到 Solitude 的 `--efu-*` 变量，所以评论表单和评论列表会随站点日间/夜间配色切换。

样式映射的核心是把 Waline 变量接到 Solitude 的主题变量上：

```css
#waline-wrap,
#waline-wrap .wl-panel {
  --waline-theme-color: var(--efu-main);
  --waline-active-color: var(--efu-main);
  --waline-white: var(--efu-card-bg);
  --waline-color: var(--efu-fontcolor);
  --waline-bg-color: var(--efu-card-bg);
  --waline-bg-color-light: var(--efu-secondbg);
  --waline-bg-color-hover: var(--efu-background);
  --waline-border-color: var(--efu-card-border);
}
```

`themes/solitude/assets/ts/comments.ts` 的 Waline 支持不是只改一个开关，主要增加了这些逻辑：

1. `requestWaline()` 请求 Waline `/api/comment`，检查 HTTP 状态和 Waline 的 `errno`。
2. `normalizeWalineRecords()` 把 Waline 的字段映射为主题通用评论结构，并将评论地址与站内文章路由匹配。
3. `fetchWalineRecentComments()`、`fetchWalinePostCardComments()`、`fetchWalineCount()` 分别取得最新评论、文章卡片评论者和评论总数。
4. `fetchPageComments()`、`fetchAggregateComments()`、`fetchPostCardComments()` 根据当前 provider 分派到 Waline 或原有 Valine 逻辑。
5. `initializeComments()` 在启用 Waline 时触发评论者头像、侧栏最新评论和评论总数渲染。

侧栏统计模板 `themes/solitude/layouts/_partials/sidebar.html` 原本只判断 Valine；现在判断 provider 时同时接受 Waline：

```go-html-template
{{ if and $cfg.comment.sidebar
  (or (in (lower $cfg.comment.use) "valine")
      (in (lower $cfg.comment.use) "waline")) }}
  <!-- 评论数量入口 -->
{{ end }}
```

移动端菜单的 CSS 选择器也收窄为 `.menus_items .menus_item > .site-page`，只给顶层菜单项应用样式，不再让 hover 规则干扰子菜单。

## 常用维护位置

| 要调整的内容 | 文件或配置 |
| --- | --- |
| 主题、语言、代码块、首页横幅、评论服务地址 | `config/pehtheme/hugo.toml` |
| 默认主题 | `config/_default/hugo.toml` |
| 自定义头图、按钮、Waline 配色、固定导航文字 | `assets/css/custom.css` |
| 封面取色及明暗模式同步 | `themes/solitude/assets/ts/covercolor/shared.ts` |
| 文章卡片评论者头像、最新评论、评论数 | `themes/solitude/assets/ts/comments.ts` |
| 侧栏组件和评论统计入口 | `themes/solitude/layouts/_partials/sidebar.html` |
| 手机轮播点击与触摸行为 | `themes/solitude/assets/ts/main.ts` |
| 首页移动端轮播链接 | `themes/solitude/assets/css/solitude/pages/home.css` |
| GitHub Pages 构建设置 | `.github/workflows/hugo.yaml` |

主题文件目前直接放在 `themes/solitude` 中，升级主题时要留意上表所列的本地定制文件，避免被主题更新覆盖。改动完成后可先用 `hugo server --environment pehtheme` 本地检查，再提交并推送：

```bash
git add . && git commit -m "Update site" && git push pages HEAD:main
```
