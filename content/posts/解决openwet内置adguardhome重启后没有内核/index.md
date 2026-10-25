---
title: 解决openwrt内置adguardhome重启后没有内核
id: 69e83616-f92c-4785-9d55-c86af8c362a6
date: 2024-06-27 08:38:11
auther: squaly
featured_image: Pasted%20image%2020261024080137.png
featured_image_preview: Pasted%20image%2020261024080137.png
summary: 不同的openwrt编译版本经常会遇到adguardhome更新内核，重启后无法启动问题， 报错：没有内核，手动更新内核后可以正常启动，但是重启后再次出现一样的问题。 解决方法： 执行文件路径设置为：/usr/bin/AdGuardHome/AdGuardHome 如果本身是本路径，可以先设置到：/
permalink: /archives/72d44c63-0f56-410c-bbab-d423453b8d91
categories:
  - network
tags:
  - openwrt
---

不同的openwrt编译版本经常会遇到adguardhome更新内核，重启后无法启动问题，

报错：没有内核，手动更新内核后可以正常启动，但是重启后再次出现一样的问题。

解决方法：

执行文件路径设置为：/usr/bin/AdGuardHome/AdGuardHome

如果本身是本路径，可以先设置到：/tmp/AdGuardHome，重启后再换回来，一般是文件夹权限问题。