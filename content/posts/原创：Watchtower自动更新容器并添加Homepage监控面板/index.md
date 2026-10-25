---
title: 原创：Watchtower自动更新容器并添加Homepage监控面板
id: f61cc82f-2431-45e5-9099-7bd914c5078f
date: 2025-10-18 10:14:00
auther: squaly
featured_image: Pasted%20image%2020261024151724.png
featured_image_preview: Pasted%20image%2020261024151724.png
summary: 近来部署的服务越来越多，经常没时间更新容器，很有必要整合一个自动更新容器的服务，于是部署了watchtower，对懒人是一大福音。 本来想用chatgpt直接写compose文件带homepage的api，结果一直出错，看来免费版的chatgpt写代码真不给力。 如果你只需要watchtower可直
permalink: /archives/LujR7Ilu
categories:
  - docker
tags:
  - docker
  - watchtower
  - homepage
---

近来部署的服务越来越多，经常没时间更新容器，很有必要整合一个自动更新容器的服务，于是部署了watchtower，对懒人是一大福音。

本来想用chatgpt直接写compose文件带homepage的api，结果一直出错，看来免费版的chatgpt写代码真不给力。

## 如果你只需要watchtower可直接用这份docker-compose文件

```
version: "3.8"

services:
  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - TZ=Asia/Shanghai                # 设置时区
      - WATCHTOWER_CLEANUP=true         # 删除旧镜像
      - WATCHTOWER_POLL_INTERVAL=3600   # 每3600秒（1小时）检查一次更新
      # - WATCHTOWER_INCLUDE_STOPPED=true  # 如果要更新已停止的容器，取消注释
      # - WATCHTOWER_LABEL_ENABLE=true     # 只更新带有特定标签的容器
      # - WATCHTOWER_NOTIFICATIONS=shoutrrr # 可选：启用通知
      # - WATCHTOWER_NOTIFICATIONS_LEVEL=info
      # - WATCHTOWER_NOTIFICATION_URL=telegram://token@telegram/?channels=@yourchannel

```

## 需要Homepage的看这里

编辑Homepage的docker-compose.yml

```yaml
version: '3.9'

services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    ports:
      - 3010:3000  # 主机 3010 -> 容器 3000（避开 Nginx 的 3000）
    volumes:
      - ./config:/app/config
      - ./logs:/logs
      - ./icons:/app/public/icons
      - ./images:/app/public/images
      - /var/run/docker.sock:/var/run/docker.sock  # (可选) Docker 配置
    environment:
      - HOMEPAGE_ALLOWED_HOSTS=ip:3010,abc.com  # 匹配 3010 端口,不需要配置域名的删除后面的,abc.com
      - PUID=1000
      - PGID=1000
      # - PORT=3000  # 默认值，可选；如需改容器内端口，取消注释并设为其他（如 3010），然后 ports: - "3010:3010"
    networks:
      - shared-net  # 新增：加入共享网络，与 Watchtower 通信
    restart: unless-stopped

networks:
  shared-net:
    external: true  # 引用外部共享网络（稍后创建）
```

### 创建共享网络

运行此命令创建网络，让 Homepage 和 Watchtower 能内部通信（无需 IP）：

```
docker network create shared-net
```

### 更新 config.yml（./config/config.yml）：yaml

```
services:
  - Watchtower:
      icon: /icons/watchtower.png
      widget:
        type: watchtower
        url: http://watchtower:8080  # 用服务名 + 容器内端口（共享网络内访问）
        key: token
```

### 步骤2: 修改 Watchtower 的 docker-compose.yml

编辑 docker-compose.yml，添加网络（其他配置保持不变）：

```
version: '3.8'

services:
  # 示例容器：默认监控所有，无需标签
  app-monitored-by-watchtower:
    image: nginx:alpine
    restart: unless-stopped
    networks:
      - shared-net

  watchtower:
    image: ghcr.io/nicholas-fedor/watchtower:latest
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_HTTP_API_TOKEN=你的token
      - WATCHTOWER_SCHEDULE=0 0 2 * * *   # 每天北京凌晨 2:00 扫描
      - WATCHTOWER_HTTP_API_PERIODIC_POLLS=true
      - TZ=Asia/Shanghai
      - WATCHTOWER_CLEANUP=true           # ← 已添加：更新后自动清理旧镜像
    command: --debug --http-api-update --http-api-metrics
    labels:
      - "com.centurylinklabs.watchtower.enable=false"
    ports:
      - "8680:8080"
    restart: unless-stopped
    networks:
      - shared-net

networks:
  shared-net:
    external: true
```

#### 使用说明

1. **默认行为**：
   * Watchtower 会扫描**所有运行容器**（Scanned=总容器数，例如您的 28 个）。
   * 如果某个容器不想监控，在其 compose.yml 中添加标签：labels: - "com.centurylinklabs.watchtower.enable=false"。
2. **验证监控**：
   * **触发立即扫描**：curl -X POST -H "Authorization: Bearer token" http://ip:8680/v1/update。
   * **查看日志**：docker logs watchtower | tail -50，应显示扫描所有容器（如 /chinesesubfinder, /metatube 等），结束时 "Scanned=XX Updated=0"。
   * **Homepage 小部件**：刷新页面，**已扫描** 会显示总容器数（例如 28），**已升级** 和 **失败** 根据实际变化。
3. **注意事项**：
   * **更新风险**：监控所有容器意味着 Watchtower 会尝试更新一切（包括数据库如 MariaDB/MySQL）。建议测试后添加排除标签给关键服务（如数据库：enable=false）。
   * **自定义排除**：如果只想监控特定容器，恢复示例标签，并为其他 compose 文件添加 enable=true。
   * **性能**：扫描 28 个容器很快（几秒），但如果容器多，可调整 --schedule 为更低频（如每天一次）。

这样配置后，Watchtower 就全覆盖了！如果日志显示意外排除，或想排除特定容器，贴出新日志我帮调\~

#### **手动扫描，通过 HTTP API 触发（最简单，立即生效）**

既然您的 Watchtower 已启用 --http-api-update，用 curl 发送 POST 请求到 /v1/update 端点，即可立即扫描所有标记 enable=true 的容器：

```
curl -X POST -H "Authorization: Bearer 你的token" http://ip:8680/v1/update
```

#### 启动Watchtower和Homepage，完结，折腾了几个小时，累成狗，以后真不能用免费gpt写代码，太不靠谱。

为了下面这个监控面板，还是值得的，毕竟Watchtower完全没ui界面，也不知道是否更新了docker，（要不查ssh日志，要不做通知推送）

![](Pasted%20image%2020261024151811.png)

