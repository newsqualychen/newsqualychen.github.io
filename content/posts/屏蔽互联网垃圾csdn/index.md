---
title: 屏蔽中文互联网有害垃圾csdn和gitcode
draft: false
featured: false
date: 2026-11-01T04:24:44+08:00
featured_image: Pasted%20image%2020261101045615.png
featured_image_preview: Pasted%20image%2020261101045615.png
categories:
  - tools
tags:
  - uBlacklist
---
不少人上网搜索资料被csdn污染搜索结果，白白浪费了很多时间，csdn的垃圾内容完全为了seo搜索而做，对查找资料完全没有帮助，实在忍无可忍。

**最推荐的方法：安装 uBlacklist 扩展（最干净、最省事）**

这是目前屏蔽 Google（以及 Bing 等）搜索结果中 CSDN 最有效、最常用的方式。

### 1. 安装扩展

1. 打开 Chrome 网上应用店，搜索 **uBlacklist**  
    直接链接：[uBlacklist - Chrome Web Store](https://chromewebstore.google.com/detail/ublacklist/pncfbmialoiaghdehhbnbhkkgmjanfhe)
2. 点击「添加至 Chrome」安装。

### 2. 添加屏蔽规则

安装后：

1. 点击浏览器右上角扩展图标 → 打开 **选项 / Options**。
2. 在规则输入框中添加以下内容（可直接复制）：

text

```
*://*.csdn.net/*
*://*.csdn.com/*
*://*.csdnimg.cn/*
*://*.gitcode.com/*
*://*.gitcode.net/*
*://*.gitcode.host/*
```

3. 点击 **Save** 保存。

> 说明：gitcode 是 CSDN 旗下产品，很多人也会一起屏蔽。

### 3. 使用效果

之后在 Google 搜索时，CSDN 相关结果会被直接隐藏。页面底部通常会显示“已屏蔽 X 个结果”，可以点击查看被屏蔽的内容。

**自定义搜索引擎方法**：

1. Chrome 设置 → 搜索引擎 → 管理搜索引擎和网站搜索 → 添加
2. 搜索引擎名称随便写，网址填：

- 按照以下内容填写表单：

- **名称**：`Google (无CSDN)`

- **快捷字串**：`google.com`

- **网址（包含查询的 URL）**：
```
{google:baseURL}search?q=%s+-site:csdn.net+-site:gitcode.com+-site:csdnimg.cn
```

- 点击保存，并在列表中找到刚刚添加的那一项，点击右侧三个点选择 **“设为默认”**。

- **效果**：

- 以后直接在 Chrome 地址栏输入关键词回车搜索时，Google 会自动在你的搜索词后加上 `-site:csdn.net`，从而把 CSDN 过滤掉。

- 设为默认搜索引擎即可。

### 4：可选：使用油猴脚本（Tampermonkey）

如果你日常使用油猴脚本管理器，可以通过脚本直接清理页面元素或重定向搜索结果。

1. 在 Chrome 中安装 **Tampermonkey（油猴）** 插件。

2. 前往 Greasy Fork 等脚本网站，搜索并安装如 **uBlacklist 增强脚本** 或支持自定义屏蔽特定域名的搜索优化脚本。

3. 在脚本配置中将 `csdn.net` 加入黑名单。

###  临时应急小技巧

如果你偶尔使用 Google，不想做任何长久配置，可以直接在搜索关键词后面手动加上 `-csdn` 或 `-site:csdn.net`。

- _例如：_ `python socket 编程 -csdn`