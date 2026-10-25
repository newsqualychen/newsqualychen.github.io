---
title: 编译openwrt官方固件
id: 019d2d17-22fa-72e4-a41a-ab775aff00fc
date: 2026-03-27 10:19:53
auther: squaly
featured_image: 
summary: 一步：不用卸载，直接新建目录 不要在原目录乱删，直接新建： cd ~mkdir openwrt-officialcd openwrt-official 第二步：拉官方源码 git clone https//github.com/openwrt/openwrt.gitcd openwrt 
permalink: /archives/LOXSXApc
categories:
tags: 
---

## 一步：不用卸载，直接新建目录

不要在原目录乱删，直接新建：

```
cd ~
mkdir openwrt-official
cd openwrt-official
```

## 第二步：拉官方源码

```
git clone https://github.com/openwrt/openwrt.git
cd openwrt
```

## 第三步：切稳定版（关键）

```
git tag
```

选最新稳定版分支

```
git switch -c build-25.12.2
```

添加源

```
vi feeds.conf.default
```

最下面添加

```
src-git openclash https://github.com/vernesong/OpenClash
src-git lucky https://github.com/gdy666/luci-app-lucky.git
```

安装指定包

```
./scripts/feeds install luci-app-openclash
./scripts/feeds install luci-app-lucky
```

选配置make menuconfig

其他设置和上篇文章一样，luci设置改动了。

Network → Web Servers/Proxies → uhttpd

[\*] uhttpd

[\*] uhttpd-mod-ubus

更新 feeds

```
./scripts/feeds clean
./scripts/feeds update -a
```

如果报错：

```
# 1. 清理旧的 feeds 缓存（重要！）
rm -rf feeds/
rm -rf package/feeds/

# 2. 更新所有 feeds（-a 表示 all）
./scripts/feeds update -a

# 3. 强制安装所有 feeds 中的包（-a -f 表示 force）
./scripts/feeds install -a -f
```
