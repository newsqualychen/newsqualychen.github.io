---
title: 实战ubuntu代替群晖做nas
id: 2340bf90-1acd-4bf7-87ca-1ee01ee94f3c
date: 2024-06-27 08:38:11
auther: squaly
featured_image: Pasted%20image%2020261024070753.png
featured_image_preview: Pasted%20image%2020261024070753.png
summary: 引言：为什么不用群晖？ .作为一个家庭服务系统，最主要的是： 1.稳定 2.故障恢复容易 3.硬件兼容性好 4.软件扩展性强 群晖很明显不具备以上几点，但这却是ubuntu的强项，linxu的开源，稳定性兼容无容置疑，唯一的缺点是使用习惯的问题而已，其实群晖的ui操作的逻辑并不好，docker的ui
permalink: /archives/79571c80-1eaf-4f4b-9fa7-08327fbb487c
categories:
  - system
tags:
  - ubuntu
---

# 引言：为什么不用群晖？

<h3>.作为一个家庭服务系统，最主要的是：</h3>

1.稳定

2.故障恢复容易

3.硬件兼容性好

4.软件扩展性强

群晖很明显不具备以上几点，但这却是ubuntu的强项，linxu的开源，稳定性兼容无容置疑，唯一的缺点是使用习惯的问题而已，其实群晖的ui操作的逻辑并不好，docker的ui设置逻辑经常变换，web station更不用说，简直是噩梦，正确设置完也不明白软件的运行原理，黑群引导升级各种痛苦（现在有aprl引导算简单了，大版本更新后仍是麻烦），硬件各种不兼容，数据恢复各种失败，经过10几年的黑群折磨后决心转向ubuntu。

## .操作步骤

2023年没人不用虚拟机了，esxi操作简单，稳定，一直是host系统首选。

### 1.直通m2硬盘

可以用多条做阵列，up主用了两条，组raid0，用来pt挂种子

### 2.安装系统

请参考网上其他教程

### 3.磁盘设置

#### 3.1 首先要su权限

```ssh
sudo passwd root  # 设置root密码
sudo su #切换root用户
```

#### 3.2 安装阵列

``` ssh
sudo apt install mdadm #root权限安装阵列软件
sudo lsblk #查看可用磁盘
```

运行结果（下面结果中的sdb和sdc两个磁盘是我们准备创建RAID 0的两块盘）：

``` ssh
NAME                MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                   8:0    0   60G  0 disk 
├─sda1                8:1    0  512M  0 part /boot/efi
├─sda2                8:2    0    1K  0 part 
└─sda5                8:5    0 59.5G  0 part 
  ├─vgubuntu-root   253:0    0 58.6G  0 lvm  /
  └─vgubuntu-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                   8:16   0   30G  0 disk 
sdc                   8:32   0   30G  0 disk 
sr0                  11:0    1 1024M  0 rom  
```

#### 3.3 创建RAID

``` ssh
root@ubuntu:~$ sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc

```

运行结果

``` ssh
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

#### 3.4 查看创建结果

``` ssh
cat /proc/mdstat
```

运行结果

``` ssh
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid0 sdc[1] sdb[0]
      62879744 blocks super 1.2 512k chunks
      
unused devices: <none>
```

从上面的输出结果中可以看到md0已经创建成功。

#### 3.5 格式化RAID

``` ssh
sudo mkfs.ext4 /dev/md0
```

运行结果

``` ssh 
mke2fs 1.45.5 (07-Jan-2020)
创建含有 15719936 个块（每块 4k）和 3932160 个inode的文件系统
文件系统UUID：1aaeec0c-d918-4e91-802e-f8f2e9645b56
超级块的备份存储于下列块： 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424

正在分配组表： 完成                            
正在写入inode表： 完成                            
创建日志（65536 个块） 完成
写入超级块和文件系统账户统计信息： 已完成 
```

挂载RAID,手动挂载命令如下（手动挂载重启后会失效，下文会介绍如何设置开机自动挂载）：

``` ssh
cd /mnt                     # 切换到/mnt路径
sudo mkdir raid0            # mnt为root权限，因此需要使用sudo来创建
sudo mount /dev/md0 raid0/
sudo chown -R squaly:squaly raid0  # 为方便使用可将目录改为当前用户权限
```

### 4 自动挂载
#### 4.1 查看当前文件系统

``` ssh
df -h
```
运行结果
``` ssh
文件系统                   容量  已用  可用 已用% 挂载点
udev                       5.9G     0  5.9G    0% /dev
tmpfs                      1.2G  3.2M  1.2G    1% /run
/dev/mapper/vgubuntu-root   58G   18G   38G   32% /
tmpfs                      5.9G   46M  5.8G    1% /dev/shm
tmpfs                      5.0M  4.0K  5.0M    1% /run/lock
tmpfs                      5.9G     0  5.9G    0% /sys/fs/cgroup
/dev/sda1                  511M  4.0K  511M    1% /boot/efi
tmpfs                      1.2G   92K  1.2G    1% /run/user/1000
/dev/md0                    59G   53M   56G    1% /mnt/raid0
```

从上面结果的最后一行可以看到我们创建的 md0 已经被挂载到了 /mnt/raid0 ，其空间大小为原始的两块磁盘的2倍。
#### 4.2 开机自动挂载RAID
查询 md0 设备的UUID，命令如下:
``` ssh
sudo blkid /dev/md0
```
运行结果：
``` ssh
/dev/md0: UUID="1aaeec0c-d918-4e91-802e-f8f2e9645b56" TYPE="ext4"
```
上面的结果中的UUID后面的字符串即为我们要获取的内容，然后通过 vim 打开配置文件：
``` ssh
sudo vim /etc/fstab
```
具体配置如下，将下面这行添加到打开的文件末尾即可：
``` ssh
UUID=1aaeec0c-d918-4e91-802e-f8f2e9645b56 /mnt/raid0 ext4 defaults 0 0
```
#### 4.3 保存RAID信息到配置文件
完成上述命令后为保证下次重启RAID配置自动生效，还需要将RAID信息保存到配置文件。
运行如下命令查看RAID详情：
``` ssh
sudo update-initramfs -u
sudo mdadm --detail --scan
```
运行结果：
``` ssh
ARRAY /dev/md0 metadata=1.2 name=kylin:0 UUID=9692c01a:85861e21:53389a0e:6b4915ad
```
参照上面配置fstab文件的方法，使用 vim 打开文件 /etc/mdadm/mdadm.conf ,将上面命令的输出结果添加到文件末尾即可。
