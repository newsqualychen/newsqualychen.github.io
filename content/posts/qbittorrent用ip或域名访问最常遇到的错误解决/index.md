---
title: qbittorrent用ip或域名访问最常遇到的错误解决
id: 233be2bb-9a5b-4f96-9266-98b9ae64d952
date: 2025-09-22 08:52:57
auther: squaly
featured_image: Pasted%20image%2020261023231504.png
featured_image_preview: Pasted%20image%2020261023231504.png
summary: 默认帐号密码 最坑是这里，新版本不是admin 和adminadmin ，网上好多这种教程，导致密码错误过多封锁ip 直接进docker查看临时密码 docker logs qbittorrent 输出最后面的是临时密码。 The WebUI administrator password was
permalink: /archives/GGWPBUBp
categories:
  - docker
tags:
  - qbittorrent
  - docker
---

### 默认帐号密码

最坑是这里，新版本不是admin 和adminadmin

，网上好多这种教程，导致密码错误过多封锁ip

直接进docker查看临时密码

```
docker logs qbittorrent
```

输出最后面的是临时密码。

```
The WebUI administrator password was not set. A temporary password is provided for this session: TYCrEFWpd
```

## 提示：unauthorized

最常见的错误，ip可以访问但用域名访问后出错，

后台-设置-webui ：启动host header（取消勾选）

![](Pasted%20image%2020261023231446.png)

## 密码验证错误或身份验证错误过多

通常出现于第三方服务调用，如果确定输入的密码是对的，但出现这个错误，直接重启qbittorrent直接解决该问题。
