---
title: 不用编译，直接用官方openwrt做软路由
id: 5952ec6d-869b-46e5-91d8-9c7b90035ab9
date: 2024-06-27 07:44:51
auther: squaly
featured_image: Pasted%20image%2020261024081051.png
featured_image_preview: Pasted%20image%2020261024081051.png
summary: 前言 玩了多年的路由系统，最初在路由器刷TOMATO，DD-WRT,梅林，后来直接装软路由，网上很多大神的版本，后来发现不满意，自己编译openwrt（&
permalink: /archives/1719445491196
categories:
  - network
tags:
  - openwrt
---

## 前言

玩了多年的路由系统，最初在路由器刷TOMATO，DD-WRT,梅林，后来直接装软路由，网上很多大神的版本，后来发现不满意，自己编译openwrt（建议github编译，省去大量的网络造成的错误），编译版本运行一年后，想升级新内核，还是需要重新编译一次，太耗时间了，而且lean大的固件对部分插件有兼容性问题，索性用官方的固件吧，安全，省心，唯一需要注意的是要自己安装插件，熟悉ssh也是很简单。

## 1 下载官方固件并安装

下载地址：https://downloads.openwrt.org/releases/ 选择最新的版本，目前下了23.05.2，Index of [*(root)*](https://downloads.openwrt.org/) / [releases](https://downloads.openwrt.org/releases/) / [23.05.2](https://downloads.openwrt.org/releases/23.05.2/) / [targets](https://downloads.openwrt.org/releases/23.05.2/targets/) / [x86](https://downloads.openwrt.org/releases/23.05.2/targets/x86/) / [64](https://downloads.openwrt.org/releases/23.05.2/targets/x86/64/) / [generic-ext4-combined-efi.img.gz](https://downloads.openwrt.org/releases/23.05.2/targets/x86/64/openwrt-23.05.2-x86-64-generic-ext4-combined-efi.img.gz)

ext4文件格式方便扩容，无论什么固件，原始空间肯定不够用，efi启动，虚拟机选什么都可以，物理机注意主板是什么启动方式，一般主板都有efi启动的。

## 2 下载完后解压在本地上传到esxi

省略上传步骤。

## 3 启动openwrt并修改len ip地址

省略

## 4 ext4磁盘扩容

参考：https://www.yydnas.cn/2023/11/2023.11.09-OpenWrt%E7%A3%81%E7%9B%98%E6%89%A9%E5%AE%B9/index.html

## 5 安装汉化包

确保网络正常的情况下运行

1、打开 OpenWrt 的管理界面，选择「System」>「Software」。

2、选择「Update lists」更新软件列表。

3、在筛选框中输入「luci-i18n-base-zh-cn」进行查找，接着选择「Install」安装语言包。

4、选择「Install」执行安装。完成后刷新页面。

## 6 安装插件

### 1 MosDNS(OpenWrt 18-23)

```ssh
opkg update
opkg install curl
```

```ssh
sh -c "$(curl -ksS https://raw.githubusercontent.com/sbwml/luci-app-mosdns/v5/install.sh)"
```

### 2 OpenClash

卸载dnsmasq，也可以在ui 页面软件包搜索，卸载。

```
opkg remove dnsmasq
mv /etc/config/dhcp /etc/config/dhcp.bak
```

这是因为接下来需要安装 `dnsmasq-full`，它会与之冲突。

```
opkg install dnsmasq-full
```

新版本基于nftables的Firewall4防火墙

```
#nftables
opkg update
opkg install coreutils-nohup bash curl ca-certificates ipset ip-full libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip kmod-nft-tproxy luci-compat luci luci-base
```

安装程序文件

在 [Github Release](https://github.com/vernesong/OpenClash/releases) 下载最新的程序文件，上传软件包进行安装。

### 3 AdGuardHome

安装 `luci-app-adguardhome`图形化管理界面

该图形化管理界面为第三方支持，Github项目地址为：[luci-app-adguardhome](https://github.com/rufengsuixing/luci-app-adguardhome)

在OpenWrt管理界面的【系统】-【软件包】，上传安装

点击【更新核心版本】，会出现以下错误提示：

```
start download
/usr/share/AdGuardHome/update_core.sh: line 168: wget-ssl: not found
download failed try another download
/usr/share/AdGuardHome/update_core.sh: line 168: wget-ssl: not found
download failed try another download
no download success
```

意思就是文件 `/usr/share/AdGuardHome/update_core.sh`的第168行，`wget-ssl`命令无法运行。

解决方法：

通过SSH连接OpenWrt，打开 `/usr/share/AdGuardHome/update_core.sh`文件，定位至168行，将 `wget-ssl`改为 `wget`：

保存后刷新界面，重新【更新核心版本】。

安装完后还有一个很坑的地方，配置文件的session ttl设置0s改成3600s，要不会一直登陆完不转跳。

## 7 安装网卡驱动

一般常用的网卡都有，如果没有你的物理网卡驱动，直接在软件包搜索，例如sriov网卡驱动，搜索：sriov

### ps:全部安装完后，记得在esxi生成快照再设置，万一以后设置乱了可以随时恢复，enjoy～～！