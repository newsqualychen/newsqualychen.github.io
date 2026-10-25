---
title: ubuntu挂载nfs文件夹
id: 9c94e089-89b8-4f7b-9321-06e0036e338f
date: 2025-09-21 04:18:30
auther: squaly
featured_image: Pasted%20image%2020261024075855.png
featured_image_preview: Pasted%20image%2020261024075855.png
summary: 查看 NFS服务器上的共享目录 显示指定的（192.168.3.1）NFS 服务器上 export 出来的目录： sudo showmount -e 192.168.3.1 创建本地挂载目录 sudo mkdir -p /mnt/data 挂载共享目录 将 NFS 服务器 192.168.3.1
permalink: /archives/a3dcOB6S
categories:
  - network
tags:
  - ubuntu
---

### **查看 NFS服务器上的共享目录**

显示指定的（192.168.3.1）NFS 服务器上 export 出来的目录：

```text
sudo showmount -e 192.168.3.1
```

### **创建本地挂载目录**

```text
sudo mkdir -p /mnt/data
```

### **挂载共享目录**

将 NFS 服务器 192.168.3.1上的 /data 目录，挂载到本地的 /mnt/ 目录下：

```text
sudo mount -t nfs 192.168.3.1:/data /mnt/data
```

采用这种方式挂载，系统重启之后不会自动挂载。

### **卸载共享目录**

卸载客户端的挂载目录，在客户端执行以下命令：

```text
umount /mnt/
```

### **共享目录的自动挂载**

把挂载信息写入到 /etc/[fstab](https://zhida.zhihu.com/search?content_id=202109222&content_type=Article&match_order=1&q=fstab&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTg1NzIwMzcsInEiOiJmc3RhYiIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjIwMjEwOTIyMiwiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.xih9pSY5MlGafO-Q-e-8tGQ9E56FsNEM8IlLg4PVJD8&zhida_source=entity) 文件中：

```text
sudo vim /etc/fstab
192.168.3.1:/data  /nfs nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
# 或者
192.168.3.1:/data  /nfs nfs defaults 0 0
```

**5.7 更新挂载**

```text
sudo mount -a
```
