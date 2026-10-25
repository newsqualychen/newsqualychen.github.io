---
title: Ubuntu 换国内源
id: 019ee846-1c44-7280-a215-7f761642fd5e
date: 2026-06-21 11:42:52
auther: squaly
featured_image: Pasted%20image%2020261024070446.png
featured_image_preview: Pasted%20image%2020261024070446.png
summary: 国内源能大幅提升 apt update 和软件下载速度。下面是通用步骤，适用于 Ubuntu 20.04/22.04/24.04 等版本。 1. 先查看你的 Ubuntu 版本和代号（Codename） Bash lsb_release -a 常见代号： 24.04 → noble 22.04 →
permalink: /archives/RfbgWavq
categories:
  - system
tags:
  - ubuntu
---

国内源能大幅提升 apt update 和软件下载速度。下面是**通用步骤**，适用于 Ubuntu 20.04/22.04/24.04 等版本。

### 1. 先查看你的 Ubuntu 版本和代号（Codename）

Bash

```
lsb_release -a
```

常见代号：

* 24.04 → **noble**
* 22.04 → **jammy**
* 20.04 → **focal**

---

### 2. 备份原源（重要！）

Bash

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

**Ubuntu 24.04+** 使用新格式，还需备份：

Bash

```
sudo cp /etc/apt/sources.list.d/ubuntu.sources /etc/apt/sources.list.d/ubuntu.sources.bak
```

---

### 3. 替换为国内源（推荐阿里云）

#### 方法一：Ubuntu 24.04 及以上（新 DEB822 格式）

Bash

```
sudo bash -c 'cat > /etc/apt/sources.list.d/ubuntu.sources << EOF
Types: deb
URIs: https://mirrors.aliyun.com/ubuntu/
Suites: noble noble-updates noble-security noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# deb-src 可选（源码）
# Types: deb-src
# URIs: https://mirrors.aliyun.com/ubuntu/
# Suites: noble noble-updates noble-security noble-backports
# Components: main restricted universe multiverse
EOF'
```

#### 方法二：传统格式（适用于 22.04 及以下，或兼容所有版本）

Bash

```
sudo bash -c 'cat > /etc/apt/sources.list << EOF
deb https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs) main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs) main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs)-updates main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs)-updates main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs)-security main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs)-security main restricted universe multiverse

deb https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs)-backports main restricted universe multiverse
deb-src https://mirrors.aliyun.com/ubuntu/ $(lsb_release -cs)-backports main restricted universe multiverse
EOF'
```

---

### 4. 更新源并升级

Bash

```
sudo apt update && sudo apt upgrade -y
```

---

### 其他国内源（替换 mirrors.aliyun.com 即可）

| 镜像             | 地址                         |
| ---------------- | ---------------------------- |
| **阿里云** | mirrors.aliyun.com           |
| **清华**   | mirrors.tuna.tsinghua.edu.cn |
| **中科大** | mirrors.ustc.edu.cn          |
| **网易**   | mirrors.163.com              |

---

**图形化换源（简单）**：

1. 打开 **软件和更新**（Software & Updates）
2. 在 **Ubuntu 软件** 标签下，**下载自** 选择 **其他...**
3. 选择 **China** → 阿里云 / 清华 等
4. 选择服务器 → **关闭** → 刷新
