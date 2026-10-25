---
title: openclaw技能开启
id: 019ce31e-fcaa-7379-a6af-565e3491946a
date: 2026-03-13 01:36:40
auther: squaly
featured_image: Pasted%20image%2020261024073108.png
featured_image_preview: Pasted%20image%2020261024073108.png
summary: 1、Agent Browser 让 AI Agent 拥有人类级的浏览器操作能力，解决了传统 AI 仅能通过 API 获取静态数据、无法直接操控浏览器的核心痛点，安装命令： npx clawhub@latest install agent-browser 它是基于Rust 开发的无头浏览器自动化 C
permalink: /archives/mhY9GnPQ
categories:
  - ai
tags:
  - open-claw
---

### 1、Agent Browser

让 AI Agent 拥有人类级的浏览器操作能力，解决了传统 AI 仅能通过 API 获取静态数据、无法直接操控浏览器的核心痛点，安装命令：

npx clawhub@latest install agent-browser

它是基于Rust 开发的无头浏览器自动化 CLI 工具，搭配 Node.js 回退功能，底层依托 Playwright/Puppeteer 实现浏览器控制，同时通过 Rust 加速提升响应速度；支持语义定位（理解 “登录按钮” 等人类化指令，而非死板匹配 HTML 标签）、状态快照（实时生成页面可访问性树）等高级能力，且采用独立无头浏览器实例运行，实现安全沙箱隔离。

覆盖网页操作全场景，支持自然语言指令直接调用，核心能力包括：

1. **网页导航与浏览**：打开任意 URL，自动获取页面元素结构化列表；
2. **表单全交互**：填写输入框、勾选复选框、选择下拉菜单、提交表单；
3. **元素操作**：单击 / 双击、拖放元素，支持文件上传至输入框；
4. **内容留存**：全页截图、录制操作过程为视频，支持 PDF 导出；
5. **高级控制**：在页面上下文执行任意 JavaScript 代码，通过 HTTP 代理访问网页（适配地理位置测试）；
6. **会话管理**：保存 Cookie 实现免密登录，多实例独立认证，跨页面保持操作状态。

### 2、Tavily Web Search

OpenClaw 的 “实时信息大脑”，联网搜索技能，让Agent能实时查最新资讯、数据，避免“闭眼编”，解决 Agent “信息滞后” 痛点，几乎所有人都说“没这个跟瞎子一样”。安装命令：

npx clawhub@latest install tavily-search

### 3、find-skills

让AI Agent自己去ClawHub搜并安装需要的技能，解决“不知道用哪个工具”的痛点。这个强烈建议大家安装！

npx clawhub@latest install find-skills

你只需告诉 Agent “我要做 XX 事”（如 “自动整理Google文档”），它会自动去 ClawHub 技能库搜索匹配的技能、推荐安装顺序、一键完成安装；

### 4、weather

OpenClaw 生态中排名前十的高频刚需技能，主打免 API 密钥、开箱即用、多格式输出，专为 AI Agent 设计，能快速响应自然语言的天气查询需求，适配自动化办公、出行规划等各类场景。

npx clawhub@latest install weather

**零门槛使用**：完全免费，无需注册、无需申请任何 API 密钥，安装后直接调用，无任何配置成本；

**双数据源保障**：集成 wttr.in 和 Open-Meteo 两大免费天气服务，避免单一数据源故障，提升查询可靠性；

### 5. self-improving-agent

内置记忆系统与自我优化机制，交互越多，能力越强。

npx clawhub@latest install self-improving-agent

核心定位：OpenClaw 的“智能进化引擎”，区别于传统固定流程自动化，实现真正的动态智能升级。

### 6、summarize

内容总结，快速消化，信息降噪神器，快速提炼核心价值，支持格式有：网页、文档（Word/PDF）、邮件、长文本、视频字幕（需搭配 OCR 技能）；

npx clawhub@latest install summarize

### 7、skill-vetter

在安装前对 ClawHub 上的技能做安全审计，识别潜在的恶意指令与风险。安装命令：

npx clawhub@latest install skill-vetter

**建议**：如果你计划频繁安装社区技能，强烈建议把它视为「隐形的第1 个必装技能」，优先级甚至可以排在所有技能之前。

### 8、Proactive Agent

给 Agent 加 “自主思考” 能力，从 “被动执行” 到 “主动规划”，给Agent加“主动性”和自我迭代能力，能记住历史、优化行为、减少重复问，长期用会很香。

clawhub install proactive-agent

### 9、gog

Google全家桶（Gmail、日历、Drive、Docs），办公自动化神器。特别是海外 / 跨境办公自动化刚需，一站式操控 Google 生态。

npx clawhub@latest install gog

**支持功能：**

* Gmail：自动收发邮件、筛选垃圾邮件、提取邮件附件 / 关键信息；
* Google 日历：自动创建日程、提醒、同步会议安排；
* Google Drive/Docs：自动新建文档、填充内容、分享权限、备份文件；

适用人群：跨境电商、海外开发者、外企办公人员，替代手动操作 Google 全家桶的重复工作；

## 更多实用Skills推荐：

#### **Github 榜单：【[点击前往](https://github.com/VoltAgent/awesome-openclaw-skills)】**

![](Pasted%20image%2020261024073052.png)

## **万能的Skills安装命令：**

npx clawhub@latest install <skill-slug>

后面<skill-slug> 改成对应的skill名称即可！

如果出现下面这个错误要多谢腾讯，直接去人家服务器拉取所有skill，导致对国内的 IP 请求频率进行了限制，等下重新输入即可。

或者直接通过 GitHub 仓库安装（手动模式）

如果 API 持续报错，你可以跳过 `clawhub` 客户端，直接利用 OpenClaw 的本地目录结构进行“伪装安装”或源码拉取：

1. 进入 OpenClaw 的技能存放目录（通常在 `~/.openclaw/skills`）。
2. 使用 `git clone` 或直接下载 `agent-browser` 的源码包。
3. 运行 `npm install` 安装依赖。

```
✖ Rate limit exceeded
Error: Rate limit exceeded
```
