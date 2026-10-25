---
title: docker安装超级好用的书签导航页Dashy，终于找到满意的了
id: 86d38415-8746-4189-a3b9-49f6739d0601
date: 2024-07-04 13:02:37
auther: squaly
featured_image: 24.21.png
featured_image_preview: 24.21.png
summary: 前言 用了n个导航书签页，终于找到个顺手，好用，美观的项目了，而且还开源 项目地址： https//github.com/Lissy93/dashy 安装方法 docker-compose.yml sudo mkdir dashy && cd dashy <
permalink: /archives/1720069357976
categories:
  - docker
  - tools
tags:
  - docker
  - dashy
---

## 前言

**用了n个导航书签页，终于找到个顺手，好用，美观的项目了，而且还开源**

![](Pasted%20image%2020261024071745.png)

### 项目地址：

[https://github.com/Lissy93/dashy](https://github.com/Lissy93/dashy)

### 安装方法

**docker-compose.yml**

```
sudo mkdir dashy && cd dashy
```

```
vim docker-compose.yml
```

```
version: "3.8"
services:
  dashy:
    image: lissy93/dashy:latest
    container_name: dashy
    restart: unless-stopped
    network_mode: bridge
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - /share/Container/dashy/conf.yml:/app/public/conf.yml  #配置文件
      - ./icons:/app/public/item-icons/icons  # 图标文件
    ports:
      - 8660:8080   #冒号前面端口随意修改
```

```
:wq   #保存
```

```
docker-compose up -d
```

### 使用方法

#### 1.1 切换语言

![](Pasted%20image%2020261024071802.png)

![](Pasted%20image%2020261024071821.png)

#### 1.2增加/删除服务

**每一个铅笔图标的地方都可以修改，所见即所得**

![](Pasted%20image%2020261024071848.png)

#### 1.3 禁用右键菜单

手机端每次滑动都弹出菜单，确实烦

![](Pasted%20image%2020261024071911.png)

### 图标设置

**Icon选项，可以设置一下几种类型**

##### 1自动获取图标

```
 icon: favicon   #icon选项填写：favicon即可使用此功能。
```

##### 2.Font Awesome 图标

**您可以使用任何**[Font Awesome 图标](https://fontawesome.com/icons)，只需指定其标识符即可。其格式为 `[category] [name]`，可在 Font Awesome 网站上该图标的页面上找到。例如：`fas fa-rocket`，`fab fa-monero`或 `fas fa-unicorn`。

**Font-Awesome 有各种各样的免费图标，但如果您有会员资格，您也可以使用其专业图标。为此，您需要在以下位置指定您的许可证密钥：**`appConfig.fontAwesomeKey`。这通常是一个 10 位字符串，例如 `13014ae648`。

##### 3.简单图标

[SimpleIcons.org](https://simpleicons.org/)是 2000 多个高质量、免费且开源的品牌和徽标 SVG 图标的集合。其用法与 font-awesome 图标非常相似。首先找到您想要在网站上使用的字形[，](https://simpleicons.org/)然后将您的图标设置为简单图标 slug，前缀为 `si-`。

```
 icon: si-portainer
```

##### 4.生成图标

**为刚刚设置的特定服务使用唯一的、以编程方式生成的图标** `icon: generative`。当您拥有大量具有不同 IP 或端口的类似服务且没有特定图标时，这尤其有用。这些图标由[DiceBear](https://api.dicebear.com/)（或[Evatar](https://evatar.io/)作为后备）生成，并使用服务域/ IP 的哈希值进行熵计算，因此每个域都将具有唯一的图标。

##### 5.表情符号图标

**您可以使用几乎任何表情符号作为项目或部分的图标。您可以直接粘贴表情符号，也可以使用其 unicode（例如** `'U+1F680'`）或短代码（例如）来指定表情符号。您可以使用[Emojipedia](https://emojipedia.org/)（靠近每个页面表情符号的底部）`':rocket:'`找到任何表情符号的这些代码，或者要快速参考表情符号短代码，请查看 @nomanoff 的[emojis.ninja](https://emojis.ninja/)。

##### 6. 直接填写URL地址的图标

```
icon: https://i.ibb.co/710B3Yc/space-invader-x256.png  #这可以是 .png、.jpg 或 .svg 格式，并托管在任何地方（本地或远程）
```

##### 7.本地图标

**直接上传到icons目录内，填写文件名就可以了，例：**

```
icon: maltrail.png    # maltrail.png就是上传到图标
```
