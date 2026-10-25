---
title: 图床利用Cloudflare的边缘缓存（CDN）加速图片访问速度
id: 26c49d84-31b6-4701-b91c-0f0adb77e4aa
date: 2024-06-30 12:53:16
auther: squaly
featured_image: Pasted%20image%2020261024075925.png
featured_image_preview: Pasted%20image%2020261024075925.png
summary: 前言： 搭建了兰空图床和Cloudflare隧道后可以实现免端口外网访问了，但是访问速度有时候比较慢，于是考虑cdn加速，看了一圈，先不
permalink: /archives/1719723196448
categories:
  - network
tags:
  - cloudflare
  - cdn
---

## 前言：

**搭建了兰空图床和Cloudflare隧道后可以实现免端口外网访问了，但是访问速度有时候比较慢，于是考虑cdn加速，看了一圈，先不说价钱，都要实名，实在麻烦，转了一圈又回到Cloudflare，本教程利用Cloudflare的边缘缓存（CDN）加速图片访问速度，而且还免费。**

## 安装准备：

### 1.安装好兰空图床，参考：

[docker搭建兰空图床及生成api](https://site.aado.top:666/archives/1719430508167)

### 2.搭建好Cloudflare隧道，参考：

[重磅功能：Cloudflare Tunnel对内网Web服务进行反向代理发布Web站点](https://site.aado.top:666/archives/1719354739654)

## 教程开始：

### 1.进入Cloudflare，你的域名-Cache Rules

### 2.新增一条规则，填写匹配域名和URI，并设置边缘TTL，笔者直接设置为1年，你可以按照自己的需求设置，最少为2小时；最后记得点击部署按钮，完成上线。

![](Pasted%20image%2020261024075951.png)

**部署完以后，再打开图片，除了首次打开速度较慢外，其他情况下基本上都是秒开，速度完全不输商业CDN。**