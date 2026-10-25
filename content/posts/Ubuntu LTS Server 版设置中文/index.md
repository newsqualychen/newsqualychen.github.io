---
title: Ubuntu LTS Server 版设置中文
id: 019ee34c-d11c-74fe-aea1-c116e3ddd8ae
date: 2026-06-20 12:32:30
auther: squaly
featured_image: Pasted%20image%2020261024070349.png
featured_image_preview: Pasted%20image%2020261024070349.png
summary: 只让 Messages（错误提示、apt 提示等）显示中文 1. 安装简体中文语言包 Bash sudo apt updatesudo apt install language-pack-zh-hans -y 2. 安装中文 man 手册（可选但推荐） Bash sudo apt install
permalink: /archives/5C8uABeH
categories:
  - system
tags:
  - ubuntu
---

### 只让 Messages（错误提示、apt 提示等）显示中文

#### 1. 安装简体中文语言包

Bash

```
sudo apt update
sudo apt install language-pack-zh-hans -y
```

#### 2. 安装中文 man 手册（可选但推荐）

Bash

```
sudo apt install manpages-zh -y
```

#### 3. 设置语言环境（关键步骤）

Bash

```
# 生成 locale
sudo locale-gen zh_CN.UTF-8

# 设置系统主要为英文 + 消息为中文
sudo update-locale LANG=en_US.UTF-8
sudo update-locale LC_MESSAGES=zh_CN.UTF-8
sudo update-locale LANGUAGE="zh_CN:zh:en"
```

#### 4. 立即在当前终端生效

Bash

```
export LANG=en_US.UTF-8
export LC_MESSAGES=zh_CN.UTF-8
export LANGUAGE="zh_CN:zh:en"
```

#### 5. 设置永久生效（推荐加入 \~/.bashrc）

Bash

```
cat >> ~/.bashrc << 'EOF'

# 系统主要英文 + 提示消息中文
export LANG=en_US.UTF-8
export LC_MESSAGES=zh_CN.UTF-8
export LANGUAGE="zh_CN:zh:en"
EOF

# 立即加载
source ~/.bashrc
```

#### 6. 重启验证

Bash

```
sudo reboot
```


### 修改系统时间（推荐方式）

系统目前使用的是 **UTC 时间**（格林威治标准时间），而实际时间应该是 **北京时间（UTC+8）**。

1. **查看当前时区**
   Bash

   ```
   date
   timedatectl status   # 如果有输出
   ```
2. **设置时区为上海（中国标准时间）**
   Bash

   ```
   sudo timedatectl set-timezone Asia/Shanghai
   ```

   **如果提示 timedatectl: command not found 或 systemd 不可用**（你的系统可能是容器环境），使用以下备用方法：
   Bash

   ```
   sudo ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
   sudo dpkg-reconfigure tzdata
   ```
3. **立即生效并验证**
   Bash

   ```
   date
   ```

**修改系统默认语言为英文（保持主要英文）**

Bash
