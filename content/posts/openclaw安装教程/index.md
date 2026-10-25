---
title: openclaw安装教程
id: 019ce23e-b1c3-70fc-a0cd-d7b638d44c7d
date: 2026-03-12 21:32:00
auther: squaly
featured_image: Pasted%20image%2020261024073144.png
featured_image_preview: Pasted%20image%2020261024073144.png
summary: 安装ubuntu桌面版-省略 进入系统设置-打开远程和修改密码 打开终端，安装ssh sudo apt update sudo apt install openssh-server sudo systemctl status ssh sudo ufw allow ssh 安装curl sud
permalink: /archives/A2HJJb3R
categories:
  - ai
tags:
  - open-claw
---

安装ubuntu桌面版-省略

进入系统设置-打开远程和修改密码

![](Pasted%20image%2020261024073209.png)

打开终端，安装ssh

```
sudo apt update
```

```
sudo apt install openssh-server
```

```
sudo systemctl status ssh
```

```
sudo ufw allow ssh
```

安装curl

```
sudo apt install curl

```

安装openclaw

```
curl -fsSL https://openclaw.ai/install.sh | bash
```

确认风险-yes

![](Pasted%20image%2020261024073237.png)

模式-快速开始

![](Pasted%20image%2020261024073256.png)

模型-选择自己的模型，第三方模型选择Custom Provider

![](Pasted%20image%2020261024073316.png)

选择openai兼容模式

![](Pasted%20image%2020261024073333.png)

选择模型id

![](Pasted%20image%2020261024073348.png)

接口命名

![](Pasted%20image%2020261024073402.png)

别名（可选）

![](Pasted%20image%2020261024073417.png)

配置通道-建议tg

![](Pasted%20image%2020261024073518.png)

填写自己的tg用户，直接填数字id

![](Pasted%20image%2020261024073534.png)

OpenClaw Agent 的搜索工具提供商选择-选择第一个

![](Pasted%20image%2020261024073549.png)

Brave Search API key-不需要，直接回车

配置技能-no,也可以选yes技能全选，需要配置api的时候选no

![](Pasted%20image%2020261024073604.png)

配置钩子-跳过

![](Pasted%20image%2020261024073624.png)

启动方式-webui

![](Pasted%20image%2020261024073646.png)

安装后自动打开浏览器进入后台

![](Pasted%20image%2020261024073703.png)

重新配置命令

```
openclaw onboard
```

重启

```
openclaw gateway restart
```

本人实测重装几次都成功完美运行，由于openclaw更新太快，教程无法一一更新，遇到新问题可留言。
