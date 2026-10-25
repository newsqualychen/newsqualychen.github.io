---
title: nastools的进化版moviepilot安装及使用教程
id: cb0142a5-834a-4485-8bd1-386de4b5f615
date: 2024-06-27 16:57:52
auther: squaly
featured_image: Pasted%20image%2020261023212540.png
featured_image_preview: Pasted%20image%2020261023212540.png
summary: 前言 玩nas的朋友基本少不了下载电影和电视剧，本人入坑黑裙也是从看电影的需求开始，以前看电影购买破解先锋蓝光机和盗版碟，&#
permalink: /archives/82f7a4d9-51eb-4d47-91cd-ba8ac2694307
categories:
  - media
tags:
  - docker
  - moviepilot
---

## 前言

玩nas的朋友基本少不了下载电影和电视剧，本人入坑黑裙也是从看电影的需求开始，以前看电影购买破解先锋蓝光机和盗版碟，越买越多，而且播放非常麻烦，每次换碟，先锋的蓝光机的机能差相应速度慢，就算接入nas也没海报墙，实在不友好。

随着片源越来越多，手动整理也是个大问题，后来买来个国产盒子，带海报墙，非常适合小白，但海报和字幕经常识别不了，plex，emby，jellyfin是不错的选择，渐渐转向了这些软件。

moviepilot就是匹配这些软件诞生，解决了自动订阅下载电视剧和电影，自动切割，下载字幕整理文件的问题，从此看片变得更舒服。

安装前必须准备好pt网站的验证用户名和token（不是密码）

## docker-compose方式安装

这份docker-compose.yml文件和其他人的不同，为了更方便使用多映射几个目录。

```
mkdir moviepilot
```

```
cd moviepilot
```

```
vim docker-compose.yml
```

```
version: '3.3'

services:

    moviepilot:
        stdin_open: true
        tty: true
        container_name: moviepilot
        hostname: moviepilot
        networks:
            - moviepilot
        ports:
            - target: 3000
              published: 3000
              protocol: tcp
        volumes:
            -  /user/downloads:/downloads  # 冒号前面改为你的下载目录,需要同下载器一样
            -  /user/video:/video #媒体库转存目录
            -  /user/docker/qbittorrent/config/qBittorrent/BT_backup:/bt #种子文件夹
            - '/moviepilot/config:/config'
            - '/moviepilot/core:/moviepilot/.cache/ms-playwright'
            - '/var/run/docker.sock:/var/run/docker.sock:ro'
        environment:
            - 'NGINX_PORT=3000'
            - 'PORT=3001'
            - 'PUID=0'
            - 'PGID=0'
            - 'UMASK=000'
            - 'TZ=Asia/Shanghai'
            - 'AUTH_SITE=iyuu'  # 这里是pt站的验证，我这里用iyuu验证支持很多pt网站
            - 'IYUU_SIGN=xxxx'  # token令牌
            - 'SUPERUSER=admin'
            - 'API_TOKEN=moviepilot'
        restart: always
        image: jxxghp/moviepilot:latest

networks:
  moviepilot:
    name: moviepilot
```

```
:wq  #保存退出
```

```
sudo docker-compose up -d  #拉娶文件，部署
```

## 安装后到设定

![](Pasted%20image%2020261023212249.png)

## 下载目录路径和整理模式

好多人选择硬链接，这里看个人需求，我的媒体库是做了raid阵列，所以我找了快m2硬盘专门pt，下载完复制到阵列硬盘，这样不会产生碎片，而且提高硬盘寿命。

### chrome浏览器安装cookiecloud插件

插件地址：https://chromewebstore.google.com/detail/cookiecloud/ffjiejobkoibkjlhjnlgmcnnigeelbdl?hl=zh-CN&utm_source=ext_sidebar

![](Pasted%20image%2020261023212305.png)

### 回到moviepilot后台填上key和密码

![](Pasted%20image%2020261023212316.png)

### 验证是否成功

![](Pasted%20image%2020261023212337.png)

### 通知设置-推荐vocechat（因为不用钱，方便而且响应速度超快）

vocechat安装教程我会放到下面链接，

新建一个机器人，名字随便填，webhook不用填，复制token填到moviepilot。

![](Pasted%20image%2020261023212419.png)


![](Pasted%20image%2020261023212429.png)

![](Pasted%20image%2020261023212440.png)

### chinesesubfinder 自动下载字幕，有需要可以下载

![](Pasted%20image%2020261023212501.png)

好了，需要用到的设置教程都写了，如果有什么地方不清晰欢迎提出～～～enjoy！

#### ps：发觉写教程还辛苦过安装，真累——！