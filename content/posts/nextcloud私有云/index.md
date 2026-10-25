---
title: nextcloud私有云
id: 310be543-4465-40c3-b9a9-334316467446
date: 2025-10-16 05:14:29
auther: squaly
featured_image: Pasted%20image%2020261023230732.png
featured_image_preview: Pasted%20image%2020261023230732.png
summary: nextcloud是一个开源的私有云文件管理系统，方便共享一些文件给小伙伴，或者协同编写文档，无需要忍受各种商业网盘龟速，而且还带app。搜索了一下网上的安装命令都不能正常运行，要不版本太旧，于是直接写了一份docker安装文件，一键安装。 docker-compose安装 version "3.
permalink: /archives/5DEJTclj
categories:
  - docker
tags:
  - nextcloud
---

nextcloud是一个开源的私有云文件管理系统，方便共享一些文件给小伙伴，或者协同编写文档，无需要忍受各种商业网盘龟速，而且还带app。搜索了一下网上的安装命令都不能正常运行，要不版本太旧，于是直接写了一份docker安装文件，一键安装。

## docker-compose安装

```
version: "3.9"

services:
  db:
    image: mariadb:11
    container_name: nextcloud_db
    restart: unless-stopped
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    environment:
      - MYSQL_ROOT_PASSWORD=your_root_password
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your_db_password
    volumes:
      - ./db:/var/lib/mysql

  redis:
    image: redis:alpine
    container_name: nextcloud_redis
    restart: unless-stopped
    command: redis-server --requirepass your_redis_password
    volumes:
      - ./redis:/data

  app:
    image: nextcloud:latest
    container_name: nextcloud_app
    restart: unless-stopped
    depends_on:
      - db
      - redis
    ports:
      - "8080:80"
    environment:
      - MYSQL_HOST=db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=your_db_password
      - REDIS_HOST=redis
      - REDIS_HOST_PASSWORD=your_redis_password
      - APACHE_SERVER_NAME=localhost
    volumes:
      - ./html:/var/www/html
     # - /mnt/nfs/nextcloudfiles:/var/www/html/data 储存盘
networks:
  default:
    driver: bridge

```

ip:8080登陆管理界面

## 域名访问

修改/html/config/config.php

```
'trusted_domains' =>
  array (
   0 => 'localhost',
   1 => 'server1.example.com',
```
