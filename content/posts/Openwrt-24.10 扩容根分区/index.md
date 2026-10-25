---
title: Openwrt-24.10 扩容根分区
id: 0d1d1524-0ada-48bc-8941-8adc4bc5bdd3
date: 2025-10-16 22:31:25
auther: squaly
featured_image: Pasted%20image%2020261024074727.png
featured_image_preview: Pasted%20image%2020261024074727.png
summary: 官方固件默认的根分区太小(默认124M)，无法安装太多软件包，通过添加新的硬盘分区达到扩容根分区的目的。 1、Openwrt安装分区软件
permalink: /archives/WNstIObp
categories:
  - network
tags:
  - openwrt
---

官方固件默认的根分区太小(默认124M)，无法安装太多软件包，通过添加新的硬盘分区达到扩容根分区的目的。

## 1、Openwrt安装分区软件

```
# 刷新仓库缓存 
opkg update
# 安装分区工具 
opkg install cfdisk fdisk e2fsprogs lsblk
# 或
opkg install parted e2fsprogs fdisk
# 备份设置
sysupgrade -b /tmp/backup.tar.gz
# 在 LuCI 网页 → 系统 → 备份/升级 → 下载备份

```

## 2、webui安装 block-mount

## 3、登录ssh为新硬盘分区

```
fdisk -l     # 列出所有磁盘
fdisk /dev/sdb  
```

新磁盘划分分区

![](Pasted%20image%2020261024074800.png)

![](Pasted%20image%2020261024074819.png)

## 4、新分区创建文件系统

```text
# 创建文件系统
mkfs.ext4 /dev/sdb1   
```

![](Pasted%20image%2020261024074836.png)

## 5、openwrt访问web管理-系统-挂载点-生成新配置，并生成配置(建议扩展 overlay分区然后重启，不折腾root分区不用再看下面了）

![](Pasted%20image%2020261024074857.png)

### 勾选已启动—作为根文件系统--最后点击保存

![](Pasted%20image%2020261024074912.png)

## 6、切换根分区

```text
mkdir -p /tmp/introot
mkdir -p /tmp/extroot
mount --bind / /tmp/introot      # 将/分区挂载到introot目录上
mount /dev/sda1 /tmp/extroot     # sda1给为sdb1
tar -C /tmp/introot -cvf - . | tar -C /tmp/extroot -xf -   # 将根分区文件复制到/dev/sdb1
umount /tmp/introot
umount /tmp/extroot
```

第四行的/dev/sda1，务必改成自己创建的新分区路径，不要照抄。

### 完成后重启openwrt

## 7、登录Openwrt终端，使用df –mT 查看根分区容量

![](Pasted%20image%2020261024074933.png)
