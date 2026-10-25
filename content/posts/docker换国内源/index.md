---
title: docker换国内源
id: 019ee831-fcd3-7276-a31b-f127609da47a
date: 2026-06-21 11:21:06
auther: squaly
featured_image_preview: Pasted image 20261023194746.png
featured_image: Pasted image 20261023194746.png
summary: Docker 切换国内源（推荐阿里云/清华大学镜像） 1. 创建/修改 Docker 配置文件 Bash sudo mkdir -p /etc/dockersudo tee /etc/docker/daemon.json > /dev/null <<EOF{  "registry-mirror
permalink: /archives/kfx0IDqq
categories:
  - docker
tags:
  - docker
---


**Docker 切换国内源（推荐阿里云/清华大学镜像）**

### 1. 创建/修改 Docker 配置文件

Bash

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.aliyun.com",
    "https://docker.mirrors.tuna.tsinghua.edu.cn",
    "https://mirror.ccs.tencentyun.com"
  ],
  "dns": ["114.114.114.114", "8.8.8.8"]
}
EOF
```

### 2. 重启 Docker 服务

Bash

```
# 重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# 查看状态确认是否正常
sudo systemctl status docker
```

### 3. **先测试能否使用**（推荐）

Bash

```
# 测试拉取镜像（用 hello-world 最快）
sudo docker run --rm hello-world
```

**或者测试常用镜像：**

Bash

```
# 测试 nginx（体积稍大，但更能验证速度）
sudo docker run --rm -p 8080:80 nginx:alpine echo "Nginx pulled successfully"
```

### 常用国内镜像地址（按速度排序推荐）：

| 镜像源          | 地址                                        | 推荐指数   |
| --------------- | ------------------------------------------- | ---------- |
| 阿里云          | https://docker.mirrors.aliyun.com           | ★★★★★ |
| 清华大学        | https://docker.mirrors.tuna.tsinghua.edu.cn | ★★★★★ |
| Docker 官方中国 | https://registry.docker-cn.com              | ★★★★   |
| 腾讯云          | https://mirror.ccs.tencentyun.com           | ★★★★   |

---

**验证当前使用的镜像源：**

Bash

```
docker info | grep -A 10 "Registry Mirrors"
```

**如果遇到权限问题**，可以把当前用户加入 docker 组：

Bash

```
sudo usermod -aG docker $USER
# 然后重新登录终端生效
```

### 最终确认（推荐再测一次大镜像速度）

Bash

```
# 测试一个稍大点的镜像，感受速度差异
sudo docker run --rm nginx:alpine echo "✅ Nginx 测试成功"
```

**删除刚刚拉取的 nginx:alpine 镜像**

### 1. 先查看当前已下载的镜像

Bash

```
docker images
```

你会看到类似这样的输出：

text

```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        alpine    xxxxxxxxxx     ...            45.2MB
hello-world  latest    ...            ...            13.3kB
```

### 2. 删除 nginx:alpine 镜像

Bash

```
# 删除指定镜像
docker rmi nginx:alpine
```

或者强制删除（推荐）：

Bash

```
docker rmi -f nginx:alpine
```

### 3. 一键清理所有未使用的镜像/容器/网络（最方便）

Bash

```
# 清理未使用的资源（推荐定期执行）
docker system prune -f
```

---

**执行完后再次确认：**

Bash

```
docker images
```

如果 nginx:alpine 已经不见了，就清理成功了。

**还原 Docker 到官方默认源（去掉国内镜像加速）**

### 方法一：彻底删除国内源配置（推荐，最干净）

Bash

```
# 1. 删除或清空 daemon.json 文件
sudo rm -f /etc/docker/daemon.json

# 或者保留文件但清空 registry-mirrors（效果一样）
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "registry-mirrors": []
}
EOF
```

Bash

```
# 2. 重启 Docker 让配置生效
sudo systemctl daemon-reload
sudo systemctl restart docker
```

Bash

```
# 3. 验证是否已还原
docker info | grep -A 5 "Registry Mirrors"
```

**正常还原后**应该只显示：

text

```
Registry Mirrors:
 Live Restore Enabled: false
```

（或者没有任何国内地址）

---

### 方法二：只注释掉或删除 registry-mirrors（保留其他配置）

Bash

```
sudo tee /etc/docker/daemon.json > /dev/null <<EOF
{
  "dns": ["114.114.114.114", "8.8.8.8"]
}
EOF
```

然后同样执行：

Bash

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

### 完整验证命令（建议执行）：

Bash

```
cat /etc/docker/daemon.json
docker info | grep -A 10 "Registry Mirrors"
```

---

**总结**：

* **想加速** → 保留带 registry-mirrors 的配置
* **想还原官方** → 删除或清空 registry-mirrors 数组，然后重启 Docker

以后需要切换时，直接改 /etc/docker/daemon.json 里的镜像列表即可。
