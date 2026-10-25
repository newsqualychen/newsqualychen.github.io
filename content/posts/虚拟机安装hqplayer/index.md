---
title: 虚拟机安装hqplayer
id: aab6d255-e4a9-4aa0-91cb-6f44709b0d7b
date: 2025-09-21 05:21:55
auther: squaly
featured_image: Pasted%20image%2020261023205404.png
featured_image_preview: Pasted%20image%2020261023205404.png
summary: 安装HQPE的一些注意事项： 转换固件 实际上装hqplayer-embedded很简单，由于img镜像不能直接安装，用StarWindV2V Converter转换成vmdk文件就可以用了。 1.1 下载好roonbox-linuxx64-nuc3-usb-factoryreset.img.gz这
permalink: /archives/ZB7Qmnlz
categories:
  - media
tags:
  - hqplayer
---
## 安装HQPE的一些注意事项：

### 转换固件

实际上装hqplayer-embedded很简单，由于img镜像不能直接安装，用StarWindV2V Converter转换成vmdk文件就可以用了。

1.1 下载好roonbox-linuxx64-nuc3-usb-factoryreset.img.gz这个文件后，用解压缩软件（如7Z、winrar等）直接解压缩，得到roonbox-linuxx64-nuc3-usb-factoryreset-100000227.img这个镜像，

1.2 同样的用StarWindV2V Converter转换成vmdk格式备用。

1.3  建立一个新的虚拟机，并添加为2个硬盘（我均选择的nvme硬盘，应该SATA也可以）

在其中一个硬盘中装载vmdk文件，提示“将现有虚拟磁盘转换为更新的格式?”，选择转换，注意此处非常奇怪，如果安装出错，不能继续使用原来的vmdk格式文件，要重新执行一次B，使用新文件才行，否则无法识别并进入通过网络安装系统的循环中。

另外一个设置为空白硬盘，建议设置大一点避免出现问题。

1.4 选择“打开电源时进入固件”，进入固件将启动顺序改为第一是加载了vmdk文件的硬盘，第二是空白硬盘；完成以后先关掉虚拟机。

1.5 虚拟机关闭状态下，在ROCK虚拟机的目录中找到后缀为vmx的配置文件，用记事本打开，添加以下内容后保存：

```
usb.restrictions.defaultAllow = "TRUE"
usb.generic.allowLastHID = "TRUE"
usb.generic.allowLastHID = "TRUE"
```

1.6 打开虚拟机，进入安装程序，在“Please type a number （1-2） and hit ENTER”处，无法输入，此时，点击工具栏“虚拟机”标签，选择“可移动设备”并点击需要使用的键盘，选择连接。注意不能使用远程控制，需要在安装虚拟机的本机上操作。

1.7 通过键盘输入选项“2”（恢复），不能选择“1”安装系统，否则后续会导致是虚拟硬盘不是SSD无法安装系统，选择“1”只能全部推倒重来。不清楚如果系统本省就是SSD硬盘是否同样有此问题。

1.8 选择恢复到空白的硬盘；选择Y即可安装完成，出现提示后，关掉虚拟机。编辑虚拟机将装有镜像的硬盘移除；

重新打开虚拟机，就会进入系统，会提示ROCK的IP地址，浏览器打开该地址，即可进入设置界面。按官网的提示，加入ffmpeg文件，更新系统即可使用。

HQPE和ROCK全部安装完成后，我们可以把不必要的设备全部 移除，例如CD/DVD，声卡等；

围绕主机，可以通过使用线电和DC-ATX电源的模式进行优化，动手能力强的还可以对主机的硬件进行更改。

### 补充说明：

虚拟机下的HQPE和ROCK都是运行在内存中的，64G内存我分配了50G给虚拟机，我升频为DSD256内存可以用满，升频至DSD512会出现明显卡顿
