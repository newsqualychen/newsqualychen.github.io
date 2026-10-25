---
title: docker安装metatube，小姐姐视频刮削元数据与图片
id: 0859c764-67fb-4ee1-87d8-55b6100c88c0
date: 2024-10-02 05:02:36
auther: squaly
featured_image: Pasted%20image%2020261024151139.png
featured_image_preview: Pasted%20image%2020261024151139.png
summary: 简介 大多数人下载av保存在特定的文件夹，里面再分子目录放不同的片源，资源一多找片非常麻烦，资源导入emby，jellyfin等媒体库后封
permalink: /archives/1719859028660
categories:
  - media
tags:
  - docker
  - emby
  - jellyfin
---

# 简介

大多数人下载av保存在特定的文件夹，里面再分子目录放不同的片源，资源一多找片非常麻烦，资源导入emby，jellyfin等媒体库后封面，简介，演员资料全无，片名也是下载文件名，非常不优雅。

有没有可能小姐姐视频也和普通电影，电视剧一样自动下载源数据呢？

github上这个开源插件出现了，解决了很多人不能优雅观看av等痛点。

## **主要原理**

+   通过`metatube-server`后端从相应数据来源爬取数据并保存在数据库中。
    
+   安装好的 MetaTube 插件通过扫描到的本地文件名，向`metatube-server`请求查询，并下载相应的元数据与图片。
    

> 其中，图像裁剪、人脸识别、自动翻译等诸多复杂的功能也均在服务端完成。

### **Docker-Compose部署**

请确保部署平台已有Docker、Docker-compose环境

```
mkdir metatube-sdk-go && cd metatube-sdk-go
curl -sL https://raw.githubusercontent.com/metatube-community/metatube-sdk-go/main/docker-compose.yml -o docker-compose.yml
docker-compose up -d  # 如果要修改端口，修改完再运行这句
```

默认端口：8080，如要修改请vim docker-compose.yml 自星修改。

## **插件安装**

## **安装之前**

请先确保当前的 Jellyfin/Emby/Plex 为**最新的**稳定版本，因为作者开发精力有限且 Jellyfin/Emby/Plex 的测试版本存在诸多不稳定性，所以本插件现在不会日后也不会提前支持这两个平台的beta版本。

在 Jellyfin/Emby/Plex 的beta测试版本中出现的插件无法识别，插件无法使用等问题，请勿在Issue中提问，亦不做任何解答，望谅解。

## **Jellyfin**

1.  进入 Jellyfin 控制台 > 插件 > 存储库，点击添加
    
2.  输入存储库名称：`MetaTube`
    
3.  输入存储库URL：`https://raw.githubusercontent.com/metatube-community/jellyfin-plugin-metatube/dist/manifest.json`
    
4.  在插件目录下找到 MetaTube，点击安装
    
5.  重启Jellyfin
    

*适用于中国大陆的存储库URL：*`https://cdn.jsdelivr.net/gh/metatube-community/jellyfin-plugin-metatube@dist/manifest.json`（可能有缓存）

## **Emby**

1.  从 [Releases](https://github.com/metatube-community/jellyfin-plugin-metatube/releases) 下载 MetaTube 最新插件
    
2.  解压出 MetaTube.dll 文件
    
3.  将 dll 文件复制到 Emby 插件目录
    
4.  重启 Emby 服务
    

PS：Emby 后续插件更新由计划任务在后台自动完成。

## **Plex**

如何找到 Plex 插件目录请参考：[https://support.plex.tv/articles/201106098-how-do-i-find-the-plug-ins-folder/](https://support.plex.tv/articles/201106098-how-do-i-find-the-plug-ins-folder/)

1.  从 [GitHub](https://github.com/metatube-community/metatube-plex-plugins/archive/refs/heads/main.zip) 或 [Releases](https://github.com/metatube-community/metatube-plex-plugins/releases) 下载最新的插件 zip 文件
    
2.  将解压后的 `MetaTube.bundle` 文件夹移入 Plex 插件目录的根目录
    
3.  其中文件夹 `MetaTubeHelper.bundle` 为辅助插件，可选择性安装
    
4.  重启 Plex 服务
    

## **目录**

常见的Emby插件目录如下：

+   群晖
    
    > /volume1/Emby/plugins
    
+   Windows
    
    > emby\\programdata\\plugins
    
+   Docker
    
    > <配置文件夹>/plugins
    > 
    > ```
    > sudo cd docker/emby/config/plugins  #进入插件目录，实际路径以自己为准，不要直接复制
    > wget https://github.com/metatube-community/jellyfin-plugin-metatube/releases/download/v2024.612.210.0/Emby.MetaTube@v2024.612.210.0.zip  #下载插件
    > unzip Emby.MetaTube@v2024.612.210.0.zip  #解压插件
    > rm -rf Emby.MetaTube@v2024.612.210.0.zip  #删除zip文件
    > ```
    
+   Linux（供参考）
    
    > /var/lib/emby-server/plugins/
    

## **emby或jellyfin设置**

### **1.插件设置****2.媒体库设置**

![](Pasted%20image%2020261023211919.png)

![](Pasted%20image%2020261023211848.png)

### **3.百度自动翻译（选装）**

api申请地址：[https://api.fanyi.baidu.com/](https://api.fanyi.baidu.com/)

登录后：首页-通用文本翻译-免费体验通用文本翻译API-开发者信息（复制APP ID和密钥到插件后台即可）
