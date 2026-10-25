---
title: docker推荐joplin笔记同步服务器安装
id: 74ee4ebe-1af4-4924-9e37-b3f8d6244e4b
date: 2024-06-28 22:54:47
auther: squaly
featured_image: Pasted%20image%2020261024152147.png
featured_image_preview: Pasted%20image%2020261024152147.png
summary: 前言： joplin是一个强大的开源笔记本软件，web，window，linux，mac，ios，android全平台都有客户端，chrome插件可以一键保存网页，支持mark，应有尽有，而&#
permalink: /archives/623f688b-769b-413e-9d21-8162c6e54ab3
categories:
  - docker
tags:
  - joplin
  - docker
---

## 前言：

joplin是一个强大的开源笔记本软件，web，window，linux，mac，ios，android全平台都有客户端，chrome插件可以一键保存网页，支持mark，应有尽有，而且自己搭建的服务用起来更放心，资料也保存在自己的服务器上，端对端加密（E2EE）。

![](Pasted%20image%2020261024152235.png)

这种选择webdav储存和服务器版的区别是笔记分享功能，如果不需要这功能可以选择webdav。

## 1 先配置cloudflare代理服务

查看我之前的教程

## 2 安装joplin服务器：

还是选择最方便的docker-compose

```
sudo mkdir joplin
```

```
cd joplin
```

```
vim docker-compose.yml
```
最重要是下面这份docker-compose，网上好多都是错的，粗心大意的人确实多。
```
version: '3'

services:
    db:
        image: postgres:15  
        volumes:
            - ./data/postgres:/var/lib/postgresql/data
        ports:
            - "5432:5432"
        restart: unless-stopped
        environment:
            - POSTGRES_PASSWORD=123456 #请修改数据库密码
            - POSTGRES_USER=joplin
            - POSTGRES_DB=joplin
    app:
        image: joplin/server:latest
        depends_on:
            - db
        ports:
            - "22300:22300"
        restart: unless-stopped
        environment:
            - APP_PORT=22300
            - APP_BASE_URL=https://a.abc.com  #域名，不用带端口，这里是唯一访问地址
            - DB_CLIENT=pg
            - POSTGRES_PASSWORD=123456 #请修改数据库密码
            - POSTGRES_DATABASE=joplin
            - POSTGRES_USER=joplin
            - POSTGRES_PORT=5432
            - POSTGRES_HOST=db

```

:wq保存后，运行程序

```
docker-compose up -d
```

## 3 进入后台

请输入域名，不要用本地ip地址+端口，前面已经填了PP_BASE_URL=你的域名，这时本地端口访问不了，会报错：

```
Invalid origin: http://*.*.*.*:22300
```

### 3.1 用正确域名进入

![](Pasted%20image%2020261024152439.png)
### 3.2 登陆后马上修改默认密码：

用户名，邮箱和地址全部修改，安全起见用admin

![](Pasted%20image%2020261024152516.png)

### 3.3 修改完后别退出，退了就再也进不去了（别问我是怎么知道的）

点击admin-emails验证邮箱

![](Pasted%20image%2020261024152613.png)

出现confirmed确认完毕就可以退出了

![](Pasted%20image%2020261024152650.png)

## 4 配置客户端

![](Pasted%20image%2020261024152723.png)

chrome插件地址：https://chrome.google.com/webstore/detail/alofnhikmmkdbbbgpnglcpdollgjjfek
完毕～～～～enjoy！