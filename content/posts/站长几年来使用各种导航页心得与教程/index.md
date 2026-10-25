---
title: 站长几年来使用各种导航页心得与教程
id: 99f5b249-8f87-49d7-938f-157ec357bf06
date: 2024-07-03 05:55:15
auther: squaly
featured_image: Pasted%20image%2020261023232216.png
featured_image_preview: Pasted%20image%2020261023232216.png
summary: 前言 玩nas时间一长必然会搭建一堆服务，特别是docker安装方便快捷，安装各种开源软件会上瘾，这时必然会搭建一个导航页方便自己
permalink: /archives/1719957315834
categories:
  - docker
  - tools
tags:
  - docker
  - homepage
---

## 前言

**玩nas时间一长必然会搭建一堆服务，特别是docker安装方便快捷，安装各种开源软件会上瘾，这时必然会搭建一个导航页方便自己存放各种服务的地址，站长尝试过n种导航页，下面说说各个导航页的优缺点。**

### 1.Heimdall

**官网：**[https://github.com/linuxserver/Heimdall](https://github.com/linuxserver/Heimdall)

![](Pasted%20image%2020261023232137.png)

**docker-compose.yml**

```
version: "3.6"
services:
  heimdall:
    image: lscr.io/linuxserver/heimdall:latest
    container_name: heimdall
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - ./config:/config
    ports:
      - 80:80
      - 443:443
    restart: unless-stopped
```

**heimdall是站长最早安装的导航页**

**优点：简洁美观，自动识别大部分服务图标，少部分手动添加。**

**缺点：也太简洁了，在pc上使用还好。一旦手机上使用你会发现排版完全是浪费，一行一个链接，手机滑动还有拖尾现象，体验差。**

**总结：只适合nas新手玩玩，服务一多完全是鸡肋。**

### 2.OneNav

**官网地址：**[https://github.com/helloxz/onenav](https://github.com/helloxz/onenav)

![](Pasted%20image%2020261023232108.png)

**docker**

```
docker run -itd --name="onenav" -p 80:80 \
    -v /data/onenav:/data/wwwroot/default/data \
    helloz/onenav:0.9.34
```

**优点：网页化书签管理，简介友好**

**缺点：在群晖安装了几次，会无缘无故挂掉，打不开网页，好几次安装好配置了网址后挂掉，太影响心情。**

**总结：不成熟，不建议安装。而且这个项目明显抄袭另外一个开源项目：Webstack**

### 3.Webstack

**官网：**[https://github.com/WebStackPage/WebStackPage.github.io](https://github.com/WebStackPage/WebStackPage.github.io)

**这是一个导航页的前端，开源搭配任意后端使用，也可以直接当静态网页使用（不建议，麻烦，维护量大）**

![](Pasted%20image%2020261023232047.png)

**站长搭配wordpress，作为wordpress的主题使用，但这延伸出另外的问题**

**优点：功能强大，舒服，爱不释手**

**缺点：备份是噩梦，玩nas一直在折腾，硬件也一直再迭更，但你打算备份迁移，你会发现工作量非常大，还不如重新填写一次链接。**

**1.旧wordpress安装备份插件，还要找破解版，备份数据，图片，主题（在不同路径）**

**2.新wordpress安装备份插件，上传备份，图片，主题。**

**3.wordpress要做反向代理，群晖安装的还要修改config文件，设置ftp权限。**

**一点都不省心啊，和部署一个博客完全没区别，有这时间不如静下心慢慢写博客算了。但这更导航页依然是我最喜欢的，效果实在好，下次用其他平台搭建后端再和大家报告。**

### 4.homepage

**又是各大博主推荐的导航页功能强大，开调用主机数据和docker接口数据，**

**官网：**[https://gethomepage.dev/latest/](https://gethomepage.dev/latest/)

![](Pasted%20image%2020261023232012.png)

**docker-compose.yml #请用root用户运行**

```
version: "3.3"
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - 3010:3000
    volumes:
      - ./config:/app/config # Make sure your local config directory exists
      - /var/run/docker.sock:/var/run/docker.sock # (optional) For docker integrations
```

**优点：严格这算是一个简易的书签系统，调用数据，api接口，key都需要配置，功能的确强大，官方也适配了大部分的软件api，照抄就可以了，适合愿意折腾的极客。**
**
上传完图标后需要关闭容器再重新打开程序才能正常载入图标，要不会显示图片错误
**
**缺点：学习成本高，部署完还要化大量时间读说明文档和修改代码。**

**总结：站长安装完估算了一下工作量，放弃了，以后又机会再尝试安装。**

### 5.flare

**官网：**[https://github.com/soulteary/docker-flare](https://github.com/soulteary/docker-flare)

![](Pasted%20image%2020261023231947.png)

**这是站长目前在用的导航页**

```
version: '3.6'

services:
  flare:
    image: soulteary/flare
    restart: always
    # 默认无需添加任何参数，如有特殊需求
    # 可阅读文档 https://github.com/soulteary/docker-flare/blob/main/docs/advanced-startup.md
    command: flare
    # 启用账号登陆模式
    command: flare --nologin=0
    environment:
      # 如需开启用户登陆模式，需要先设置 `nologin` 启动参数为 `0`
      # 如开启 `nologin`，未设置 FLARE_USER，则默认用户为 `flare`
      - FLARE_USER=flare
      # 指定你自己的账号密码，如未设置 `FLARE_USER`，则会默认生成密码并展示在应用启动日志中
      - FLARE_PASS=flare
      # 是否开启“使用向导”，访问 `/guide`
      - FLARE_GUIDE=1
    ports:
      - 3020:5005
    volumes:
      - ./app:/app
```

**优点：安装简单，页面简洁，编辑简单。**

**缺点：图标设置非常麻烦，需要到另外的页面挑选图标，而且一定没有你想要的图标，只能找差不多的代替，造成选择服务的时候往往不知道在哪里？要慢慢找，不带后台登录，映射公网后每个人都可以看到。做反向代理稍微安全点，但也不是很放心。**

**总结：临时使用，过渡性产品。**

**Q&A**

**那么多导航页到底那个最好？**

**世界上没完美的人，也没有完美的程序，目前使用下来还是Webstack和homepage比较适合，Webstack尝试用其他后端看看能不能解决备份麻烦的问题，或者一步到胃学习homepage。站长尝试后再告诉大家，其他差不多的导航页不用试了，导航页是为了使用舒服，其次才是好看。**


### 6.Dashy

nashy可以满足所有需求，有兴趣请移步到我刚刚写的文章：

[docker安装超级好用的书签导航页Dashy，终于找到满意的了](https://site.aado.top:666/archives/1720069357976 "docker安装超级好用的书签导航页Dashy，终于找到满意的了")

![](Pasted%20image%2020261023231920.png)
