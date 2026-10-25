---
title: emby利用jellyfin安装Theme Songs自动刮削片头曲-全网首发，转载请注明出处
id: 31e5a2b4-d84e-447c-af2c-33c6f51ab452
date: 2024-07-02 08:16:14
auther: squaly
featured_image: Pasted%20image%2020261023210151.png
featured_image_preview: Pasted%20image%2020261023210151.png
summary: 前言 相信用过netfilx的朋友对它的片头动画和片头曲特别有印象，用户体验实在高级，有没有emby和jellyfin可以实现的插件？答案是有的，可&#
permalink: /archives/1719879374582
categories:
  - media
tags:
  - emby
---

## 前言

**相信用过netfilx的朋友对它的片头动画和片头曲特别有印象，用户体验实在高级，有没有emby和jellyfin可以实现的插件？答案是有的，可是emby上面要收费，由于中国人贫穷的特质，很少人愿意花钱，而且更变态的是emby的插件作者把电影和电视剧分开卖，片头曲和动画也分开，5刀一个，4个一起买有优惠一共10欧元（78人仔）。**

![](Pasted%20image%2020261023210103.png)

![](Pasted%20image%2020261023210114.png)


**付费版效果可以的，可是站长家境贫穷，只好选择开源免费软件。**

### 插件：Jellyfin Theme Songs Plugin

**官网：**[https://github.com/danieladov/jellyfin-plugin-themesongs](https://github.com/danieladov/jellyfin-plugin-themesongs)

### 安装方法

1. **在 jellyfin 中，转到仪表板 -> 插件 -> 存储库 -> 添加并粘贴此链接**[https://raw.githubusercontent.com/danieladov/JellyfinPluginManifest/master/manifest.json](https://raw.githubusercontent.com/danieladov/JellyfinPluginManifest/master/manifest.json)
2. **进入目录并搜索主题曲**
3. **单击它并安装**
4. **重启 Jellyfin**

## 设置

### 1.jellyfin设置

**此插件依赖TVDB刮削,后台必须把元数据下载器TVDB勾上，我是用moviepilot刮削的，jellyfin只是用来刮削主题曲，所以我没勾上其他元数据下载器，如果你主用jellyfin的可以勾上其他。**

**自动从互联网获取元数据并刷新，我这里选择：从不，如果你本身没刮削过的请选择手动刷新一次。**

![](Pasted%20image%2020261023210130.png)

### 2.jellyfin后台

**计划任务- Theme Songs手动运行一次任务，（其实是自动运行的，第一次手动可以马上刷新下载）。**

### 3.前台设置

#### 1.jellyfin点击用户-显示，勾选主题曲（如果你安装jellyfin只是为了下载主题曲可以跳过此步）

#### 2.emby后台-显示，勾上启用主题曲。（app，tv客户端也是够上主题曲就可以了）

**完毕～～enjoy～～～！**