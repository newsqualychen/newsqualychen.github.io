---
title: 从零开始搭建并发布 Hugo 博客：FixIt、Obsidian 与 GitHub Pages
draft: false
featured: false
date: 2026-10-25T02:38:35+08:00
featured_image: Pasted%20image%2020261025183615.png
featured_image_preview: Pasted%20image%2020261025183615.png
categories:
  - blog
tags:
  - hugo
---

这套博客采用静态网站生成方案。文章使用 Markdown 编写，由 Hugo 生成网页，再通过 GitHub Actions 自动构建并发布到 GitHub Pages。
<!--more-->
## 一、整体方案
整个流程如下：

```
Obsidian / 文本编辑器
          │
          ▼
   Markdown 文章与图片
          │
          ▼
   Hugo 项目源文件
          │
          ▼
      Git 提交
          │
          ▼
      GitHub 仓库
          │
          ▼
   GitHub Actions 自动构建
          │
          ▼
    GitHub Pages 发布
          │
          ▼
       浏览器访问
```

如果有多台设备，可以将 Hugo 项目放在 NAS 共享目录中，让 Mac 和 Linux 电脑访问同一套源文件。手机和平板也可以通过支持访问共享目录的编辑方式修改文章。

需要注意：共享目录负责存储和编辑，Git 负责版本管理，GitHub Actions 负责构建和发布。这三个环节各自承担不同的职责。

## 二、准备所需软件

主要需要以下软件：

|软件|用途|
|---|---|
|Hugo Extended|将 Markdown 和模板生成静态网站，并支持主题需要的扩展功能|
|Git|管理文章、主题和配置的版本|
|Obsidian|编写和管理 Markdown 文章|
|FixIt|Hugo 博客主题|
|GitHub|保存博客源文件|
|GitHub Actions|自动构建和部署网站|
|GitHub Pages|托管生成的网站|

Hugo Extended 与普通版本并不完全相同。使用主题之前，应先确认主题所需的 Hugo 版本及 Sass 编译方式。

官方文档：

- [Hugo 安装指南](https://gohugo.io/installation/)
    
- [Hugo 快速入门](https://gohugo.io/getting-started/quick-start/)
    
- [Hugo GitHub Pages 部署指南](https://gohugo.io/host-and-deploy/host-on-github-pages/)
    

## 三、安装 Hugo

### 3.1 macOS

如果使用 Homebrew，可以先尝试：

```
brew install hugo
```

安装后检查版本：

```
hugo version
```

如果主题要求 Extended 版本，应确认输出中包含 `extended`。如果软件包版本不符合项目要求，应根据 Hugo 官方安装页面选择合适的发行版。

### 3.2 Fedora Linux

可以先查看系统是否已经安装 Hugo：

```
hugo version
```

如果尚未安装，可以使用 Fedora 软件仓库：

```
sudo dnf install hugo
```

安装后再次检查：

```
hugo version
```

如果仓库中的版本太旧，或者不是主题所需的 Extended 版本，可以从 [Hugo 官方发布页面](https://github.com/gohugoio/hugo/releases)下载对应 Linux 架构的发行包。

不要只看命令是否能够运行，还应检查版本号和发行版类型。

### 3.3 Windows

Windows 可以使用 WinGet 安装 Hugo Extended：

```
winget install -e --id Hugo.Hugo.Extended
```

安装后重新打开终端并执行：

```
hugo version
```

如果 Windows 终端暂时找不到 `hugo` 命令，可以先关闭并重新打开终端，再检查环境变量配置。

## 四、创建 Hugo 项目

如果还没有项目，可以使用 Hugo 创建一个新站点：

```
hugo new site my-blog
cd my-blog
```

如果已经有现成的 Hugo 项目，则不需要重新创建，也不要覆盖现有配置。

Hugo 项目通常包含以下目录：

```
my-blog/
├── archetypes/
├── assets/
├── config/
│   └── _default/
├── content/
├── layouts/
├── static/
├── themes/
├── hugo.toml
└── .gitignore
```

不同主题和 Hugo 版本生成的目录结构可能有所不同。

常见目录用途：

- `content/`：文章、分类页和其他内容。
    
- `config/`：网站及主题配置。
    
- `themes/`：主题文件。
    
- `static/`：直接复制到生成网站中的静态文件。
    
- `assets/`：需要 Hugo 处理的资源。
    
- `layouts/`：自定义页面模板。
    
- `public/`：Hugo 构建生成的网站文件，通常不提交到源代码仓库。
    

## 五、安装和配置 FixIt 主题

FixIt 是一个适用于 Hugo 的主题。安装方式应以 [FixIt 官方项目说明](https://github.com/hugo-fixit/FixIt)为准。

如果使用 Git 子模块管理主题，可以参考以下命令：

```
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
```

如果主题已经存在，不要重复添加子模块。

主题安装完成后，按照 FixIt 的文档配置网站参数。常见配置包括：

- 网站标题和语言。
    
- 网站时区。
    
- 菜单与导航。
    
- 文章列表布局。
    
- 代码高亮。
    
- 图片与封面。
    
- 评论、搜索等可选功能。
    

配置文件可能位于 `hugo.toml` 或 `config/_default/` 下，具体以项目现有结构为准。

### 关于 Sass 编译

部分 Hugo 主题需要 Dart Sass 处理 SCSS 样式。

如果构建时出现 Sass 编译错误，应先检查：

1. Hugo 是否为主题要求的版本。
    
2. Dart Sass 是否正确安装。
    
3. 当前构建环境是否能够找到 Sass 可执行文件。
    
4. 主题使用的 Sass 接口是否与安装版本兼容。
    

不要在没有确认原因的情况下反复安装不同的 Sass 实现。应优先遵循主题和 Hugo 的官方说明。

## 六、配置网站地址

如果使用 GitHub Pages 的项目网站，地址通常类似：

```
https://你的用户名.github.io/仓库名/
```

在 Hugo 配置文件中设置对应的 `baseURL`，例如：

```
baseURL = "https://你的用户名.github.io/仓库名/"
```

这里的用户名和仓库名需要替换成自己的实际值。

`baseURL` 非常重要。设置不正确时，网站可能出现 CSS、JavaScript、图片或页面链接无法加载的问题。

如果以后改用自定义域名，应重新检查 `baseURL`、域名 DNS 和 GitHub Pages 的域名设置。

## 七、在 Obsidian 中编写文章

### 7.1 打开已有项目

如果 Hugo 项目已经存放在本地磁盘或 NAS 共享目录中，Obsidian 应直接打开这个已有文件夹作为仓库，而不是再新建一份空仓库。

例如，项目结构可能是：

```
hugo/
└── content/
    └── posts/
        └── my-first-post/
            ├── index.md
            └── image.png
```

具体路径以自己的设备和共享目录挂载方式为准。

如果 Mac 和 Fedora 都访问同一份 NAS 文件，应避免两台电脑同时对同一 Git 仓库执行提交、切换分支或拉取等操作，以免出现锁文件或工作区冲突。

### 7.2 创建文章

可以先建立一个文章目录，再在其中创建 `index.md`：

```
content/posts/my-first-post/index.md
```

文章内容示例：

```
---
title: "我的第一篇文章"
date: 2026-10-25T12:00:00+08:00
draft: false
description: "这是一篇 Hugo 测试文章。"
---

## 开始

这里是文章正文。

这是一张图片：

![示例图片](image.png)
```

如果图片与 `index.md` 放在同一个目录中，正文可以使用相对路径引用图片。

建议文章目录采用页面资源包的形式，并使用标准的 `index.md` 文件名。不要误命名成 `index.md.md`。

### 7.3 图片管理

图片可以放在文章目录内：

```
my-first-post/
├── index.md
└── image.png
```

也可以统一存放到 `static/` 目录下，再通过网站根路径引用。

两种方式都可以，但路径规则不同，不要混用。

如果 Obsidian 自动生成的图片文件名包含空格，必须确认 Markdown 引用路径与真实文件名一致，并在本地构建后检查生成的 HTML 和图片文件。

### 7.4 摘要设置

Hugo 不要求每篇文章都必须手动填写 `summary` 字段。

可以使用自动摘要，也可以在正文中使用 Hugo 的摘要分隔标记：

```
<!--more-->
```

标记前的正文可作为文章摘要。但首页最终是否只显示摘要，还取决于主题模板是否调用 Hugo 的摘要字段。

如果 FixIt 首页显示了全文，应检查主题的文章列表模板，而不是仅仅添加一个空的 `summary` 字段。

## 八、本地预览网站

在 Hugo 项目根目录执行：

```
hugo server
```

终端会显示本地预览地址，通常是：

```
http://localhost:1313/
```

用浏览器打开地址即可预览网站。

修改文章后，Hugo 通常会自动重新生成预览内容。

需要检查的项目包括：

- 首页文章列表是否正常。
    
- 文章标题和日期是否正确。
    
- 图片是否显示。
    
- 菜单链接是否有效。
    
- 手机屏幕下的布局是否正常。
    
- 文章摘要是否符合预期。
    

本地预览正常并不代表线上一定正常，因此发布后还需要检查 GitHub Actions 构建结果。

## 九、使用 Git 管理源文件

### 9.1 检查仓库状态

在 Hugo 项目根目录执行：

```
git status
```

如果项目已经关联 GitHub 仓库，可以检查远程仓库：

```
git remote -v
```

远程地址应指向自己拥有或有权限推送的仓库。

### 9.2 配置忽略文件

在项目根目录的 `.gitignore` 中，可以忽略 Hugo 自动生成的文件：

```
/public/
/resources/
/.hugo_build.lock
.DS_Store
```

注意：不要把自己需要版本管理的配置、文章、图片或主题文件误加入忽略列表。

如果主题使用 Git 子模块，必须同时保留 `.gitmodules` 文件及相应的子模块记录。

### 9.3 首次提交

如果项目还没有进行过 Git 提交，可以执行：

```
git add .
git commit -m "Initial Hugo site"
```

如果远程仓库尚未配置，需要根据自己的 GitHub 仓库地址添加远程仓库，并确认分支名称与部署工作流一致。

### 9.4 后续更新

每次完成文章修改后：

```
git status
git add .
git commit -m "Update blog posts"
git push
```

如果推送失败，应先检查分支、远程地址、身份验证和仓库权限，不要直接删除 Git 元数据或强制覆盖远程仓库。

## 十、配置 GitHub Pages 自动部署

推荐采用 GitHub Actions 构建和部署的方式。这样 GitHub 保存的是 Hugo 源文件，而不是手动上传生成的网页文件。

官方说明：

[Hugo 官方 GitHub Pages 部署文档](https://gohugo.io/host-and-deploy/host-on-github-pages/)

### 10.1 创建 GitHub 仓库

在 GitHub 上创建一个用于存放 Hugo 项目源文件的仓库。

将本地 Hugo 项目提交并推送到该仓库。

注意：仓库中应包含构建所需的文章、配置、主题和相关资源；通常不需要提交 `public/` 目录。

### 10.2 配置 Pages

进入 GitHub 仓库：

1. 打开 `Settings`。
    
2. 进入 `Pages`。
    
3. 将发布来源设置为 `GitHub Actions`。
    

### 10.3 添加工作流

在项目根目录创建：

```
.github/workflows/hugo.yaml
```

工作流负责完成以下工作：

1. 检出 GitHub 仓库的源代码。
    
2. 安装与项目匹配的 Hugo Extended。
    
3. 安装主题需要的 Dart Sass 等依赖。
    
4. 执行 Hugo 构建。
    
5. 上传 `public/` 生成目录。
    
6. 将构建结果部署到 GitHub Pages。
    

工作流必须与实际分支名一致。例如，若项目使用 `master` 分支，自动触发条件就不能只监听 `main` 分支。

同时要确认 Git 子模块能够被完整检出，否则构建环境可能找不到主题文件。

如果主题依赖 Sass，工作流必须安装正确的 Dart Sass。仅在自己的电脑上安装 Sass，并不会自动让 GitHub Actions 也拥有该工具。

### 10.4 检查部署结果

提交并推送工作流文件后：

1. 打开 GitHub 仓库的 `Actions` 页面。
    
2. 找到最新的构建任务。
    
3. 查看构建与部署日志。
    
4. 确认任务成功完成。
    
5. 打开 GitHub Pages 地址，检查实际页面。
    

如果任务失败，优先查看日志中最早出现的错误。常见原因包括 Hugo 版本不兼容、主题子模块未检出、Sass 依赖缺失、配置错误或仓库权限不足。

## 十一、日常发布流程

网站部署成功后，日常更新可以简化为：

```
1. 使用 Obsidian 编辑文章
2. 保存 Markdown 与图片
3. 本地运行 hugo server 检查
4. 使用 Git 查看改动
5. 提交并推送到 GitHub
6. 等待 GitHub Actions 自动构建
7. 检查线上网站
```

不需要每次都手动运行 `hugo` 生成 `public/`，因为 GitHub Actions 可以负责正式构建。

如果只是修改了文章，却没有推送到 GitHub，线上网站不会自动更新。

## 十二、常见问题

### 1. 首页显示整篇文章

检查 FixIt 的文章列表模板是否使用 `.Content`。如果希望只显示摘要，应确认模板使用 `.Summary` 或正确处理摘要分隔标记。

### 2. 图片在 Obsidian 中显示，线上却不显示

检查：

- 图片是否存在于项目源文件中。
    
- 图片是否已经提交并推送。
    
- Markdown 引用路径是否正确。
    
- 文章和图片是否处于预期的页面资源包目录。
    
- 图片是否出现在 Hugo 构建生成的目录中。
    

### 3. 本地可以构建，GitHub Actions 失败

本地环境和 GitHub Actions 环境可能安装了不同版本的软件。检查 Hugo、主题、Dart Sass 及 Git 子模块是否一致。

### 4. 网站样式丢失或链接异常

检查 `baseURL`、GitHub Pages 的项目路径，以及主题生成的静态资源地址。

### 5. Git 推送失败

检查远程仓库地址、分支名称、身份验证和权限。如果工作流文件需要推送到仓库，也应确认当前认证方式具备相应权限。

### 6. 多台设备编辑出现冲突

同一份 NAS 文件适合共享编辑，但不代表可以在多台设备上同时进行 Git 操作。建议一次只在一台设备上执行提交和推送，完成后再切换设备。

## 总结

Hugo 的优势是文章使用纯文本 Markdown 保存，网站可以由自动化流程生成和发布。通过 Obsidian 编写文章、Git 管理源文件、GitHub Actions 自动构建，以及 GitHub Pages 托管，可以建立一套跨设备维护、方便备份和迁移的博客工作流。

最重要的是把内容、版本管理和部署三个环节分开：文章保存在源文件中，Git 记录改动，自动化工作流负责发布。这样后续更换设备、主题或托管平台时，也更容易维护和迁移。