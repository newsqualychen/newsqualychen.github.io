---
title: 绅士必备软件Tissue纸巾
id: 561f0906-6537-4b8e-a809-ad0e571ec906
date: 2025-09-22 17:55:42
auther: squaly
featured_image: Pasted%20image%2020261024153012.png
featured_image_preview: Pasted%20image%2020261024153012.png
summary: 自从nastool升级成moviepilot后，麻烦的事情也增加了，小姐姐不能自动从下载盘复制整理到储存盘，只能用脚本或手动复制，太麻烦而且不优雅。有没有更方便的方式整理小姐姐的资源？于是出现了Tissue，一个自动下载，刮削，整理的工具。 docker-compose安装 version "3.
permalink: /archives/YOl79rY9
categories:
  - media
tags:
  - docker
---

自从nastool升级成moviepilot后，麻烦的事情也增加了，小姐姐不能自动从下载盘复制整理到储存盘，只能用脚本或手动复制，太麻烦而且不优雅。有没有更方便的方式整理小姐姐的资源？于是出现了Tissue，一个自动下载，刮削，整理的工具。

## docker-compose安装

```
version: "3.7" # 最新版该行可删除

services:
  tissue:
    image: chris2s/tissue:latest
    container_name: tissue
    restart: unless-stopped
    environment:
      - TZ=Asia/Shanghai
    ports:
      - "9193:9193" # 冒号左侧可改
    volumes:
      - ./config:/app/config # 实际配置文件存储路径，冒号左侧替换，下同
      - /media:/data/video # 媒体库，刮削完毕后的存放位置
      - /path/for/file:/data/file # 可作为本项目还不支持qb以外下载器替代方案，即
将本地址映射到其他下载器的下载路径
      - /download:/downloads # qb下载路径，会自动扫描
```

## 使用方法

WEB 输入 NAS_IP:9193，即可访问到应用界面。

默认用户名：admin

默认密码：password

首页直接有小姐姐的资源免费下载，这里不放图了

![](Pasted%20image%2020261024153158.png)

## 设置

设置-通用，检查一下路径：`/data/video`就是存放的目的地

![](Pasted%20image%2020261024153128.png)


文件 路径：/downloads（qbittorrent安装时的默认路径）

重点说说下载

填写qbittorrent的地址，用户名和密码

下载路径/downloads后面再加个文件夹，例如：hm，免得和其他下载的电影混在一起

勾上自动转移。

![](Pasted%20image%2020261024153058.png)
