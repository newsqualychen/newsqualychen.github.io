---
title: emby容器推荐amilys-embyserver
id: e3079bbb-c281-42ea-a112-4a061a4c8401
date: 2024-06-27 08:38:11
auther: squaly
featured_image: null
summary: title emby容器推荐amilys/embyserver cover https//ls.aaaz.top83/i/2023/11/14/655339608b5ed.png tags emby abbrlink 46998 date 2023-11-09 023947 toc
permalink: /archives/52be0427-885d-472e-bf8b-68887c9738dc
---

---
title: emby容器推荐amilys/embyserver
cover: https://ls.aaaz.top:83/i/2023/11/14/655339608b5ed.png
tags: emby
abbrlink: 46998
date: 2023-11-09 02:39:47
toc:
toc_number:
---

# 前言

开心版的emby容器有很多，比较出名的是lovechan，zishuo（不推荐，已经3年多没更新了），下面介绍的是amilys，目前使用比较满意，主要推荐原因是集成了几个插件：

容器地址：

https://registry.hub.docker.com/r/amilys/embyserver/

## 1:增加了主题：

https://github.com/Nolovenodie/emby-crxhttps://github.com/Nolovenodie/emby-crx

![主题](https://photo.aaza.top:666//i/2023/11/28/6565d6b122442.png)

## 2:dd-danmaku Emby 弹幕库插件（这插件我一直没成功开启过）：

https://github.com/RyoLee/dd-danmaku

开启/关闭 请在/config/config/ext.sh 中设置

![S0](https://ls.aaaz.top:83/i/2023/11/11/654f3199a81f7.png)

## 3：调用外部播放器：

https://github.com/bpking1/embyExternalUrl

### 重点推荐

非常实用的插件，本地播放基本上不需要显卡转码，现在的硬件足够强大了，根本不需要显卡转码，tv播放，up主用<b>shield tv 2019 pro</b>电视上直接源码播放，主要是网页chrome和手机上播放，部分视频没转吗直接播放不了，现在有了这个插件，完美解决了。

服务端安装后，在浏览器会出现播放器的图标，点击图标就可以调用外部播放器了。 

![emby](https://ls.aaaz.top:83/i/2023/11/09/654c6adc63520.png)

<b>vlc需要修改路径，mac用户推荐使用iina</b>

## 4.修改了tdmb的dns

不用魔法上网也可以正常获得源数据

# 安装方式：(可选)



## docker-compose安装（推荐）

```
version: '3.5'
services:
    emby:
      image: amilys/embyserver:latest
      container_name: emby
      network_mode: bridge
      volumes:
        - ./config:/config
        - ./cache:/cache
        - /mnt:/mnt
      ports:
        - '8096:8096'
      devices:
        - /dev/dri:/dev/dri
      privileged: true
      environment:
        - UID=0
        - GID=0
        - GIDLIST=0
        - TZ="Asia/Shanghai"
      extra_hosts:
        - "api.themoviedb.org:143.204.82.126"
        - "image.tmdb.org:99.86.7.63"
        - "themoviedb.org:99.84.130.66"
```