---
title: Ubuntu 升级到最新版本及设置ESM Apps
id: 019ede72-9359-76cc-a54f-90c82a3bed69
date: 2026-06-19 13:55:18
auther: squaly
featured_image: Pasted%20image%2020261024070535.png
featured_image_preview: Pasted%20image%2020261024070535.png
summary: Ubuntu 22.04.5 LTS 升级到最新版本 1. 升级前准备（非常重要！） 备份数据：使用 Timeshift、Deja Dup 或手动备份重要文件、/home 目录和配置文件。 更新当前系统： Bash sudo apt update && sudo apt upgrade -ysud
permalink: /archives/OAimzqZ8
categories:
  - system
tags:
  - ubuntu
---

Ubuntu 22.04.5 LTS 升级到最新版本

### 1. 升级前准备（非常重要！）

* **备份数据**：使用 Timeshift、Deja Dup 或手动备份重要文件、/home 目录和配置文件。
* 更新当前系统：
  Bash

  ```
  sudo apt update && sudo apt upgrade -y
  sudo apt dist-upgrade -y
  sudo apt autoremove -y
  ```
* 重启系统：
  Bash

  ```
  sudo reboot
  ```
* 检查当前版本：
  Bash

  ```
  lsb_release -a
  ```

### 2. 推荐升级方法：使用 do-release-upgrade

#### 选项 A：升级到最新可用 LTS（推荐）

Bash

```
# 安装升级工具（如果没有）
sudo apt install update-manager-core -y

# 检查是否有可用升级
sudo do-release-upgrade -c

# 开始升级（正常模式）
sudo do-release-upgrade
```

### 注意事项

* **升级过程** 可能需要 30-90 分钟，取决于网络和硬件。期间不要断电或中断。
* 如果是 **服务器**（无桌面），推荐用 do-release-upgrade。
* 第三方 PPA、Docker、NVIDIA 驱动等可能需要升级后手动修复。
* 如果升级卡住，可尝试编辑 /etc/update-manager/release-upgrades，将 Prompt=lts 改为 Prompt=normal，然后重试。
* 升级失败常见原因：磁盘空间不足、第三方源冲突、特定包问题。先清理系统再试。

升级过程中会：

* 下载大量软件包（几百MB到几GB）
* 询问是否继续（输入 y）
* 可能需要处理配置文件冲突（一般选择 **保留当前版本** 或 **安装维护者版本**，看情况）
* 多次重启

### 注意事项：

* **升级时间**：视网络速度而定，通常 30\~90 分钟。
* **不要中断**：中途断电或关闭终端可能会导致系统无法启动。
* 如果你是**桌面版**（有图形界面），也可以直接打开 **软件更新器**（Software Updater），它会提示升级并提供图形化向导。
* 升级完成后，建议运行以下命令清理系统：
  Bash

  ```
  sudo apt update && sudo apt upgrade -y
  sudo apt autoremove -y
  ```

**启用 ESM Apps（Expanded Security Maintenance for Applications）**

Ubuntu 22.04 LTS 默认只对 **Main** 仓库提供 5 年安全更新，而 **Universe** 仓库中的大量应用（比如很多常用软件）只能获得有限支持。启用 **ESM Apps** 可以让这些应用额外获得**最长 10 年** 的安全更新。

### （免费个人订阅，最多 5 台机器）

1. **在浏览器中打开这个链接**（推荐用你当前这台机器的浏览器）：
   **[https://ubuntu.com/pro](https://ubuntu.com/pro)**
2. **使用 Ubuntu One 账号登录**（如果没有就注册一个，免费）。
3. 登录后，点击 **Get Ubuntu Pro** 或直接进入仪表盘，选择 **Personal / Free** 订阅（免费，最多 5 台机器）。
4. 你会看到一个 **长字符串的 Token**（类似一串随机字符），**复制它**。
5. **回到终端，执行下面命令**（把 <你的token> 替换成复制的那串）：

Bash

```
sudo pro attach <你的token>
```

示例：

Bash

```
sudo pro attach abcdef1234567890abcdef1234567890
```

---

### 绑定成功后

Bash

```
sudo pro status
```

你应该看到 esm-apps 已经 **enabled**。

然后刷新软件源：

Bash

```
sudo apt update
```
