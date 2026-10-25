---
title: 完全个人的Spotify-开源本地音乐整理及声纹音乐苦分析项目
id: 01a05e77-f215-7100-8890-6388aa02a474
date: 2026-09-02 03:35:22
auther: squaly
featured_image: Pasted%20image%2020261024153353.png
featured_image_preview: Pasted%20image%2020261024153353.png
summary: AudioMuse-AI 的核心作用是基于音频本身的声学特征（如节奏、音色、能量、旋律结构）进行 AI 深度分析，而不是单纯依赖传统的 ID3 标签（流派、艺术家、专辑）。 完成全库分析与聚类后，你将获得以下实际使用效果： 1. Navidrome 客户端无缝扩展 一键生成 Instant Mix（
permalink: /archives/QSCdMafN
categories:
  - media
tags:
  - navidrome
  - audiomuse
---

AudioMuse-AI 的核心作用是**基于音频本身的声学特征（如节奏、音色、能量、旋律结构）进行 AI 深度分析**，而不是单纯依赖传统的 ID3 标签（流派、艺术家、专辑）。

完成全库分析与聚类后，你将获得以下实际使用效果：

**1. Navidrome 客户端无缝扩展**

* **一键生成 Instant Mix（即时混合歌单）**：在 Navidrome 或第三方 App（如 Symfonium、Feishin）中点击任意一首歌的“即时混合/相似曲目”，AudioMuse 会基于声学指纹实时推荐曲风、听感极其相似的歌曲。
* **以曲拓人（Artist Radio / Song Radio）**：打破传统标签限制，即使是完全不同流派的歌手，只要音频气场相似也会被算法拉进同一个电台。

**2. 2D 交互式“音乐地图”（Music Map）**

* 打开 AudioMuse Web 端的 **Music Map** 界面，你的整个 Navidrome 音乐库会被可视化为一张散点星空图。
* 距离越近的歌曲，听感和情绪越相似。你可以直接在地图上圈选特定区域，批量导出为随心听歌单。

**3. 音频过渡与“音乐路径”（Song Paths / Transition）**

* 指定起点歌曲 A（如：安静的古典乐）和终点歌曲 B（如：高燃的重金属），AudioMuse 会从你的曲库中挑选中间过渡歌曲，自动生成一条**曲风平滑渐变、节奏循序渐进**的音乐路径歌单。

**4. 自动生成声学主题歌单（Audio-based Clusters）**

* 聚类完成后，后台会自动梳理出多个无需人工打标签的“氛围歌单”（例如：适合专注的工作背景音、高能量运动电台、夜间低沉 Chill 歌单），极大地激活曲库里被冷落的冷门好歌。

Navidrome 是一个开源的基于 Web 的音乐收藏服务器和流媒体播放器。它让您可以自由地通过任何浏览器或移动设备收听您的音乐收藏。它就像您的个人 Spotify！

AudioMuse-AI 是一款开源且可自行托管的工具，它利用声音分析来重新发现音乐库中被遗忘的歌曲，并生成能够捕捉每首曲目背后含义的律动感知播放列表，而无需依赖元数据或外部 API。

您可以使用 Docker Compose 或 Podman 在本地运行它，也可以将其大规模部署到 Kubernetes 集群（支持**AMD64**和**ARM64 ），或者使用适用于****macOS、Windows 和 Linux 的原生应用程序。它已与包括**[Navidrome](https://www.navidrome.org/)、[Jellyfin](https://jellyfin.org/)、[LMS](https://github.com/epoupon/lms/tree/master)、[Lyrion](https://lyrion.org/)、[Emby](https://emby.media/)和[Plex](https://www.plex.tv/)在内的主流自托管音乐服务器集成，并且计划集成更多服务。

经过几天的调试,完美运行,下吗直接上干货.docker完装.

```
services:
  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    user: "3000:3000"
    ports:
      - "4533:4533"
    restart: unless-stopped
    environment:
      ND_LOGLEVEL: info
      ND_SCANSCHEDULE: "@every 24h"
      ND_SESSIONTIMEOUT: "24h"
      # --- AudioMuse / 插件配置 ---
      ND_PLUGINS_ENABLED: "true"
      ND_PLUGINS_AUTORELOAD: "true"
      ND_AGENTS: "audiomuseai,lastfm,deezer"
      # 连接同网络下的 AudioMuse API 服务
      ND_AUDIOMUSE_URL: "http://audiomuse-ai-flask:8000"
    volumes:
      - "./data:/data"
      - "./plugins:/data/plugins"
      - "/mnt/nfs/media/music:/music:ro"
    depends_on:
      - audiomuse-ai-flask

  # AudioMuse 专用的 PostgreSQL 数据库
  audiomuse-postgres:
    image: postgres:15-alpine
    container_name: audiomuse-postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: audiomuse
      POSTGRES_PASSWORD: audiomusepassword
      POSTGRES_DB: audiomusedb
    volumes:
      - audiomuse-db-data:/var/lib/postgresql/data

  # AudioMuse 主 Web/API 服务 (Flask)
  audiomuse-ai-flask:
    image: ghcr.io/neptunehub/audiomuse-ai:latest
    container_name: audiomuse-ai-flask-app
    restart: unless-stopped
    ports:
      - "8000:8000" # AudioMuse 控制台入口，访问地址 http://<服务器IP>:8000
    environment:
      SERVICE_TYPE: "flask"
      POSTGRES_USER: audiomuse
      POSTGRES_PASSWORD: audiomusepassword
      POSTGRES_DB: audiomusedb
      POSTGRES_HOST: audiomuse-postgres
      POSTGRES_PORT: "5432"
      TEMP_DIR: "/app/temp_audio"
    volumes:
      - temp-audio-flask:/app/temp_audio
      - plugins-flask:/app/plugin/installed
      - "/user/music/(你的音乐库根目录):/music:ro" # 挂载相同的音乐文件目录
    depends_on:
      - audiomuse-postgres

  # AudioMuse 后台分析工作线程 (Worker)
  audiomuse-ai-worker:
    image: ghcr.io/neptunehub/audiomuse-ai:latest
    container_name: audiomuse-ai-worker-instance
    restart: unless-stopped
    environment:
      SERVICE_TYPE: "worker"
      POSTGRES_USER: audiomuse
      POSTGRES_PASSWORD: audiomusepassword
      POSTGRES_DB: audiomusedb
      POSTGRES_HOST: audiomuse-postgres
      POSTGRES_PORT: "5432"
      TEMP_DIR: "/app/temp_audio"
    volumes:
      - temp-audio-worker:/app/temp_audio
      - plugins-worker:/app/plugin/installed
      - "/user/music/(你的音乐库根目录):/music:ro" # 挂载音乐文件目录以进行 AI 算法分析
    depends_on:
      - audiomuse-postgres

volumes:
  audiomuse-db-data:
  temp-audio-flask:
  temp-audio-worker:
  plugins-flask:
  plugins-worker:
```

运行后安装插件

安装与配置步骤

* **下载插件**：从 [AudioMuse-AI-NV-plugin Releases](https://github.com/NeptuneHub/AudioMuse-AI-NV-plugin/releases) 页面下载 `audiomuseai.ndp` 文件。
* **放置文件**：将 `audiomuseai.ndp` 放入 Navidrome 的数据插件目录中（默认路径通常为 `/data/plugins` 或 `navidrome/plugins/`）。
* **重启服务**：重启 Navidrome 服务以加载插件(http://ip:4533)。
* **后台启用**：登录 Navidrome 管理后台，进入 **UI -> Plugins**，启用 AudioMuse-AI 插件。
* **填写配置**：设置 AudioMuse-AI 的 API URL 地址以及其他相关配置参数（如 API Token 等）本compose文件直接填写地址为:http://audiomuse-ai-flask:8000
* 保存后在插件页面启动插件.
* 登陆AudioMuse:http://ip:8000,添加媒体服务器,Navidrome,设置Navidrome的地址,根据前面的 `docker-compose.yml` 配置，请按照以下参数修改：
* **AudioMuse-AI API URL**: `http://audiomuse-ai-flask:8000`
* **API Token**: 留空即可（除非你在 AudioMuse 中单独设置了 Token）
* **Server Name**: `navidrome`（对应上一步在 AudioMuse Web 界面里填写的 **Display name** 名称，填入后方便多服务器映射）
* 用户名密码是Navidrome的登陆密码.
* 勾选 **Set as the default server**
* **配置 Authentication 账号**：
* * **Enable authentication**: 保持 `True`
  * **Admin Username**: 设置你的 AudioMuse **管理员用户名**（
  * **Admin Password**: 设置你的 AudioMuse **管理员密码**
  * **Confirm Admin Password**: 再次输入确认密码
  * **JWT Secret**: 保持默认自动生成的字符串即可（无需修改）
  * **API token (optional)**: 留空即可（如果刚才在 Navidrome 插件端填了 Token，可以保持一致，一般默认不填）
* **提交保存**：
  * 滚动到页面**最底部**，点击 **Save Configuration** / **Complete Setup** 类似颜色的蓝色提交按钮。
* **进入面板**：
  * 提交完成后，页面会跳转到登录框（输入刚才设置的 Admin 账号密码）或直接进入控制面板 Dashboard。

AudioMuse-AI设置

### 第一步：启动音频特征分析（Analysis）

1. 在 AudioMuse 的左侧导航栏或主菜单中，找到并点击 **Analysis and Clustering**（分析与聚类）页面。
2. 在 **Analysis**（音频分析）卡片区域，点击 **Start Analysis** 按钮。
3. **后台运行机制**：
   * 此时 `audiomuse-ai-worker` 容器开始接管任务，从 Navidrome 中拉取歌曲，并读取挂载的音乐文件路径 `/music`。
   * 它会对音频波形进行本地神经网络运算，提取声学指纹（Sonic Fingerprint）。
   * 页面上会显示进度条（例如 `Launched`, `Completed`, `Skipped` 等）。

> **提示**：根据音乐库的大小和 CPU/GPU 算力，首次全库分析可能需要数小时到上百分钟。你可以随时通过 **Pause/Stop** 按钮暂停，后续随时重新启动，它会从断点继续提取未分析的歌曲。

### 第二步：运行声音聚类（Clustering）

当全库（或大部分常用歌曲）分析完成后，才能开启聚类：

1. 在同一个页面下方的 **Clustering** 区域。
2. 保持默认参数（或根据需求调整聚类生成的歌单数量）。
3. 点击 **Run Clustering**（运行聚类）。
4. 系统会将声学指纹相似的歌曲自动归类分组，生成不依赖传统流派/标签（Genre）的声学主题歌单（Audio-based Playlists）。

### 第三步：生成与体验成果

聚类与分析完成后，AudioMuse 的以下核心功能将正式解锁：

1. **Music Map（音乐地图）**： 在左侧菜单进入** Music Map**，你可以看到一张 2D 的音乐星空图，音频特征相似的歌曲会散布在相近的位置。
2. **Instant Playlists / Song Paths（智能歌单与过渡路径）**： 可以指定两首不同风格的歌曲，让 AI 自动挑选过渡歌曲生成顺畅切换的歌单。
3. **Navidrome 前端无缝联动**： 由于你已经在 Navidrome 中配置好了插件，此时直接在 Navidrome 的 Web 端或支持 Subsonic 的客户端（如 Symfonium、AmuseNet 等）中，对任意一首歌右键选择 **Instant Mix（即时混合/相似歌曲）**，Navidrome 就会实时向 AudioMuse 请求由 AI 算法生成的同频曲目列表。

更多高级玩法:

连接Last.fm和listenbrainz

配置方法（修改 docker-compose.yml）

在你的 Navidrome 服务环境变量段落（environment）下，直接追加以下两行：

```
YAML
    environment:
      # ... 原有的其他环境变量 ...
      ND_LASTFM_APIKEY: "你的Lastfm_API_Key"
      ND_LASTFM_SECRET: "你的Lastfm_Secret"
修改保存后执行 sudo docker compose up -d 重新创建容器即可生效。
```

在 Navidrome 中配置全局的 `ND_LASTFM_APIKEY` 和 `ND_LASTFM_SECRET` 后，它主要负责**为全站获取封面图、歌手简介和相似歌手数据**。

对于**多用户使用**以及**个人听歌记录（Scrobble / 刷歌）**，具体机制如下：

* **刮削元数据（全局生效）**：所有用户在浏览 Navidrome 时，都能共享由这个全局 API Key 抓取到的封面、背景图和歌手信息。
* **个人听歌打卡 / Scrobble（按用户隔离）**：如果不同用户希望将自己的听歌历史同步到各自的 Last.fm 个人账号，**不需要**在 Docker 环境变量里做隔离，而是由**各个用户在各自的客户端或前端单独绑定**。

**多用户配置个人 Last.fm 账号的方法：**

1. **在 Navidrome 网页端绑定**：
   * 每个用户使用自己的账号登录 Navidrome Web 界面。
   * 点击右上角个人头像 -> **Personal**（个人设置）。
   * 在 **Last.fm** 区域点击 **Connect to Last.fm**，授权登录各自的 Last.fm 账号即可。
2. **在第三方播放客户端绑定**：
   * 如果用户使用 Symfonium、Feishin、Subtracks 等客户端，可以直接在客户端设置中开启 Last.fm Scrobble，并登录各自的账号。

这种架构下，全局的 API Key 负责获取基础设施数据，而用户个人的 Auth 授权决定记录刷到谁的账户上，互不干扰。
