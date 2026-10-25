---
title: ubuntu基础配置
id: 731805a7-140f-401a-87c5-1ba85e2d7a96
date: 2025-10-03 15:32:14
auther: squaly
featured_image: Pasted%20image%2020261024070628.png
featured_image_preview: Pasted%20image%2020261024070628.png
summary: ubuntu安装后添加一些必要的基本配置：更新，nfs，docker 这里有个坑，虚拟机安装的时候别选liunx和sriov网卡，直接选择ubuntu64位和vm网卡，要不安装的时候有可能报错，sriov网卡可能在down状态。 安装nfs服务器 sudo apt updatesudo apt u
permalink: /archives/PC946j8k
categories:
  - system
tags:
  - ubuntu
---

ubuntu安装后添加一些必要的基本配置：更新，nfs，docker

这里有个坑，虚拟机安装的时候别选liunx和sriov网卡，直接选择ubuntu64位和vm网卡，要不安装的时候有可能报错，sriov网卡可能在down状态。

安装nfs服务器

```
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
sudo apt install nfs-kernel-server
```

更新完会出现警告

![](Pasted%20image%2020261024070701.png)

| 类别                    | 服务名称                                                                                                                           | 说明                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| **网络/文件系统** | nfs-blkmap-dispatcher.service nfs-mount.service rpcbind.service rpc-stat.service systemd-networkd.service systemd-resolved.service | NFS 和 RPC 相关，网络文件共享。             |
| **系统核心**      | polkit.service systemd-journald.service systemd-logind.service systemd-manager systemd-timesyncd.service                           | 权限管理、日志、登录、时间同步等核心。      |
| **虚拟化/VM**     | open-vm-tools.service vgauth.service                                                                                               | VMware 虚拟机工具（如果在 VM 中运行）。     |
| **存储/磁盘**     | udisks2.service                                                                                                                    | 磁盘管理。                                  |
| **SSH/远程**      | ssh.service                                                                                                                        | SSH 服务（如果启用）。                      |
| **用户/升级**     | unattended-upgrades.service[user@1000.service](mailto:user@1000.service)                                                              | 自动升级和用户会话（@1000 是你的用户 ID）。 |

### 如何操作？

用空格全部打上*，全部安装。

运行 sudo apt autoremove 清理无用包。

挂载nfs，在另外的文章有写，不再重复

安装docker

`添加 Docker 库`

```sudo
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
```


然后，运行下列命令添加 Docker 的官方 GPG 密钥：

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

`添加 Docker 官方库：`

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

`使用命令更新 Ubuntu 源列表：`

```
sudo apt update
```


### 安装 Docker

`最后，运行下列命令在 Ubuntu 22.04 LTS 服务器中安装最新 Docker CE`

```
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

`安装完成后，运行如下命令验证 Docker 服务是否在运行：`

```
sudo systemctl status docker
```

输出

```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2025-10-03 10:16:21 UTC; 8s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 20042 (dockerd)
      Tasks: 13
     Memory: 21.5M
        CPU: 1.195s
     CGroup: /system.slice/docker.service
             └─20042 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

`使 Docker 服务在每次重启时自动启动：`

```
sudo systemctl enable docker
```

`查看已安装的 Docker 版本：`

```
sudo docker version
```

## 安装 Docker Compose

`安装最新稳定的 Docker Compose 文件：`

```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

如果有更新版本，只需要将上述命令中的 **v2.6.1** 替换为最新的版本号即可。请不要忘记数字前的 **"v"** 。

`最后，使用下列命令赋予二进制文件可执行权限：`

```
sudo chmod +x /usr/local/bin/docker-compose
```

`运行下列命令检查安装的 Docker Compose 版本：`

```
sudo docker-compose version
```
