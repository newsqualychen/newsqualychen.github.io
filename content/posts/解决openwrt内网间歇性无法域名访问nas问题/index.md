---
title: 解决openwrt内网间歇性无法域名访问nas问题
id: 0089c8b6-bbc5-4535-95db-bd202a62ec6e
date: 2024-06-27 08:38:11
auther: squaly
featured_image: Pasted%20image%2020261024080041.png
featured_image_preview: Pasted%20image%2020261024080041.png
summary: 引言 使用双软路由经常会出现域名访问内网的环路问题，具体情况是域名可以ping通ip但是却间歇性不能访问，遇到这种问题请一一排查原因。 1. 终端ping域名 1.1 如果不能ping通，在软路由继续ping，出现错误ip地址的话是软路由的缓存造成，，重启软路由或负责缓存的软件， 下面一个个排查，缓
permalink: /archives/cea4bf06-42ae-4bc2-8d4e-7b7d8ab29cc1
categories:
  - network
tags:
  - openwrt
---

# 引言

使用双软路由经常会出现域名访问内网的环路问题，具体情况是域名可以ping通ip但是却间歇性不能访问，遇到这种问题请一一排查原因。
## 1. 终端ping域名
### 1.1 如果不能ping通，在软路由继续ping，出现错误ip地址的话是软路由的缓存造成，，重启软路由或负责缓存的软件，
下面一个个排查，缓存不是越多越好，按照习惯开一个就可以了：
* Turbo ACC 网络加速设置，关闭dns缓存。
* 关闭martDNS或MosDNS。
* 清除AdGuard Home缓存。
* 双路由爱快不要设置dns缓存，openwrt缓存dns更专业.
### 1.2 如果可以ping通
openwrt-网络，主机名：域名 ip地址：内网转发的ip（通常为群晖ip），设置完清除浏览器缓存或用无痕模式测试即可。