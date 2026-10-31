---
title: 小雅Alist-TVBox-Emby等一键安装及更新元数据自动同步
id: 8f214429-2348-4a10-aaa1-f7392d2ddbaf
date: 2024-07-04 04:22:14
auther: squaly
featured_image: Pasted%20image%2020261023205735.png
featured_image_preview: Pasted%20image%2020261023205735.png
summary: 前言 去年开始流行起将阿里云盘的影视流同步到alist，于是有人想到emby使用WebDAV连接alist播放视频 经过前人不断的尝试，引申出更
permalink: /archives/1720038133870
categories:
  - media
tags:
  - emby
  - alist
---
2025年更新：alist已经闭源，请改用openlist，阿里云网盘已经限制播放，小雅项目仅作为参考，套进其他可以用的网盘。
## 前言

**去年开始流行起将阿里云盘的影视流同步到alist，于是有人想到emby使用WebDAV连接alist播放视频**

**经过前人不断的尝试，引申出更快更便捷的安装脚本，以下脚步可以做到一键安装alist，emby，自动清除缓存等。**

**但安装过程中尽量别断开ssh，直至完成所有交互命令。**

![](Pasted%20image%2020261023205632.png)

![](Pasted%20image%2020261023205644.png)

### 1.安装前准备：

**1.阿里云盘中至少还有70G的空间**

**2.nas有160g以上空间，用来存放元数据**

**3.nas需要有docker环境**

**4.三个必要参数:**

|                    | **对应文件**                                       | **获取方式**                                                                                                                                                                                                                                                                                                              |
| ------------------ | ---------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **token**          | **/etc/xiaoya/mytoken.txt**                    | [https://aliyuntoken.vercel.app/](https://aliyuntoken.vercel.app/) [https://alist.nn.ci/zh/guide/drivers/aliyundrive.htm](https://alist.nn.ci/zh/guide/drivers/aliyundrive.htm)l                                                                                                                                      |
| **open token**     | **/etc/xiaoya/myopentoken.txt**                | [https://alist.nn.ci/zh/guide/drivers/aliyundrive\_open.html](https://alist.nn.ci/zh/guide/drivers/aliyundrive_open.html)                                                                                                                                                                                             |
| **转存目录的folder id** | **/etc/xiaoya/temp\_transfer\_folder\_id.txt** | **先转存这个**[https://www.aliyundrive.com/s/rP9gP3h9asE](https://www.aliyundrive.com/s/rP9gP3h9asE)到自己网盘（选择资源盘），然后浏览器打开转存后的目录，浏览器的url[https://www.aliyundrive.com/drive/file/resource/640xxxxxxxxxxxxxxxxxxxca8a](https://www.aliyundrive.com/drive/file/resource/640xxxxxxxxxxxxxxxxxxxca8a)最后一串就是，记得这个目录不要删，里面的内容可以定期删除 |

### 2.安装步骤：

**1.ssh进入主机并切换root权限**

**2.ubuntu安装curl 命令（群晖一般不用安装）**

```
sudo apt install curl
```

**3.拉取安装脚本**

```
bash -c "$(curl --insecure -fsSL https://ddsrem.com/xiaoya_install.sh)"
```

### 3.功能列表

**数字代表先选x，再选x，再选x**

**PS: 2 2 8代表先选2，再选2，最后选8**

**普通功能**

```
———————————————————————————————————————安装———————————————————————————————————————
安装 小雅Alist -> 1 1
安装 小雅Alist-TVBox -> 5 1
安装/更新 小雅助手（xiaoyahelper）-> 4 1
安装 Onelist -> 6 1
安装 Portainer -> 8 1 1
安装 Emby全家桶（一键） -> 2 1
安装 Jellyfin全家桶（一键） -> 3 1
安装 Resilio-Sync（单独） -> 2 5 1
安装 Auto_Symlink -> 8 2 1
安装 CasaOS -> 8 4 1
安装 小雅元数据定时爬虫 -> 2 9 1
——————————————————————————————Emby手动全家桶配置————————————————————————————————————
单独 下载并解压 全部元数据 -> 2 2 1
单独 解压 全部元数据 -> 2 2 2
单独 下载 all.mp4 -> 2 2 3
单独 解压 all.mp4 -> 2 2 4
解压 all.mp4 的指定元数据目录【非全部解压】-> 2 2 5
单独 下载 config.mp4 -> 2 2 6
单独 解压 config.mp4 -> 2 2 7
单独 下载 pikpak.mp4 -> 2 2 8
单独 解压 pikpak.mp4 -> 2 2 9
选择 下载器【aria2/wget】-> 2 2 10
单独 安装Emby（可选择版本，支持官方，amilys，lovechen）-> 2 3
立即 同步小雅Emby的config目录 -> 2 6
单独 创建/删除 同步定时更新任务 -> 2 7
图形化编辑 emby_config.txt -> 2 8
————————————————————————————Jellyfin手动全家桶配置——————————————————————————————————
单独 下载并解压 全部元数据 -> 3 2 1
单独 解压 全部元数据 -> 3 2 2
单独 下载 all_jf.mp4 -> 3 2 3
单独 解压 all_jf.mp4 -> 3 2 4
解压 all_jf.mp4 的指定元数据目录【非全部解压】-> 3 2 5
单独 下载 config_jf.mp4 -> 3 2 6
单独 解压 config_jf.mp4 -> 3 2 7
单独 下载 PikPak_jf.mp4 -> 3 2 8
单独 解压 PikPak_jf.mp4 -> 3 2 9
选择 下载器【aria2/wget】-> 3 2 10
单独 安装Jellyfin-> 3 3
———————————————————————————————————————更新———————————————————————————————————————
更新 小雅Alist-TVBox -> 5 2
更新 小雅Alist -> 1 2
更新 Resilio-Sync（单独） -> 2 5 2
更新 Onelist -> 6 2
更新 Portainer -> 8 1 2
更新 Auto_Symlink -> 8 2 2
更新 小雅元数据定时爬虫 -> 2 9 2
———————————————————————————————————————卸载———————————————————————————————————————
卸载 小雅Alist -> 1 3
卸载 Emby全家桶 -> 2 10
卸载 卸载Jellyfin全家桶 -> 3 4
卸载 Resilio-Sync（单独） -> 2 5 3
卸载 小雅助手（xiaoyahelper）-> 4 3
卸载 小雅Alist-TVBox -> 5 3
卸载 Onelist -> 6 3
卸载 Portainer -> 8 1 3
卸载 Auto_Symlink -> 8 2 3
卸载 CasaOS -> 8 4 2
卸载 小雅元数据定时爬虫 -> 2 9 3
————————————————————————————————Docker Compose—————————————————————————————————————
安装 小雅及全家桶 -> 7 1
卸载 小雅及全家桶 -> 7 2
——————————————————————————————————————系统工具——————————————————————————————————————
查看系统磁盘挂载 -> 8 3
———————————————————————————————————————其他———————————————————————————————————————
一次性运行 小雅助手（xiaoyahelper）-> 4 2
创建/删除 定时同步更新数据（小雅alist启动时拉取的数据）-> 1 4
```

**高级功能**

```
Docker启动容器名称设置 -> 9 1
是否开启容器运行额外参数添加 -> 9 2
重置脚本配置 -> 9 3
开启/关闭 磁盘容量检测 -> 9 4
开启/关闭 小雅连通性检测 -> 9 5
Docker镜像源选择 -> 9 6
```

### 4.使用小技巧

由于片源实在太多，在tv端搜索有些麻烦，可在手机端先找到喜欢的片源，tv端在最爱，快速找到资源。

### 体验总结：

阿里云盘-alist-emby这种方式联动播放视频，优缺点也很明显。

优点：不占硬盘，免费白嫖各种影视资源

缺点：

1.利用流的形式播放视频，播放不能暂停时间太长。

2.不能继续播放。

3.没有跳过片头，等高级功能，

4.片源读取需要载入等待（站长的宽带1000m下载，载入速度可以接受，但体验远没本地好）