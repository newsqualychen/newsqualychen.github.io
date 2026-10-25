---
title: docker安装emby开心版教程
id: d62c20fc-59db-4703-99cd-70f7adfa1763
date: 2024-06-30 09:47:37
auther: squaly
featured_image: Pasted%20image%2020261023212056.png
featured_image_preview: Pasted%20image%2020261023212056.png
summary: 简介： emby版本有很多，为什么选择amilys版本？这本比起lovechen版本更新更快，站长安装一年多测试下来没任何问题，可以映射显卡解码，安装小雅也正常使用，另外这个版本带tmdb，themoviedb的代理很方便，推荐使用。 docker-conpose安装 sudo mkdir emby
permalink: /archives/1719712057376
categories:
  - docker
tags:
  - emby
---

## 简介：

emby版本有很多，为什么选择amilys版本？这本比起lovechen版本更新更快，站长安装一年多测试下来没任何问题，可以映射显卡解码，安装小雅也正常使用，另外这个版本带tmdb，themoviedb的代理很方便，推荐使用。

### docker-conpose安装

```
sudo mkdir emby
```

```
cd emby
```

```
vim docker-compose.yml
```

```
version: '3.5'
services:
    emby:
      image: amilys/embyserver:latest
      container_name: emby2
      network_mode: bridge
      volumes:
        - ./config:/config
        - ./cache:/cache
        - /mnt/usr/video:/video   # 这里是影音文件地址，修改冒号前面为自己的路径
      ports:
        - '8096:8096'    # 端口，冒号前面随意修改
      devices:
        - /dev/dri:/dev/dri         # 显卡驱动，一般不用修改
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

```
docker-compose up -d
```

完毕。

emby使用简单，实在没什么好说的，后期再介绍些高级使用技巧。