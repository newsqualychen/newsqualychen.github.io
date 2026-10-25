---
title: Nginxproxymanager端口转发教程
id: b7dd4312-356f-4d7a-a7da-f88b3ed2941d
date: 2024-06-27 08:38:11
auther: squaly
featured_image: 20261024080314.png
featured_image_preview: 20261024080314.png
summary: 被Nginxproxymanager折腾了很久，网上的教程只教了http转发和ssl，但是内网是没有443端口的，导致好多人只能转发http，https转发失败，下面直接给出docker-compose安装文件，
permalink: /archives/390cd209-e33f-4d47-a7ab-f95bed7a729b
categories:
  - network
tags:
  - nginx
  - docker
---

被Nginxproxymanager折腾了很久，网上的教程只教了http转发和ssl，但是内网是没有443端口的，导致好多人只能转发http，https转发失败，下面直接给出docker-compose安装文件，直接一键安装就可以了，所有教程就是卡在这里，默认的443端口没有修改，但我们家用是被封443端口的所以要修改其他，不修改的话一直访问失败。

## 1 docker-compose直接安装

```
version: '3'
services:
  app:
    image: 'chishin/nginx-proxy-manager-zh:release'
    restart: always
    ports:
      - '80:80'
      - '81:81'
      - '668:443' #左边668为映射的443端口，可随便修改
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

## 2 如果已经安装了的不用重装，用portainer进行修改就可以了，
在容器详情-复制编辑，按下图修改

![](Pasted%20image%2020261024080812.png)

## 3 Nginxproxymanager设置，记得设置泛域名*.aaa.com这种形式。
### 3.1 获取cloudflare的API
首先获取cloudflare账户API的TOKEN。
登录到cloudflare账户，然后查看My Profile，点击Global API Key的View API Key，输入密码，保存KEY。
### 3.2 填上域名和TOKEN

![](Pasted%20image%2020261024080835.png)

### 3.3 新建代理服务

![](Pasted%20image%2020261024080852.png)

![](Pasted%20image%2020261024080955.png)

## 4 路由器映射端口
前面已经把Nginxproxymanager的443端口映射成668，只需要在路由器，映射出端口就成功了。

![](Pasted%20image%2020261024081014.png)

