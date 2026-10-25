---
title: openwrt备份及恢复
id: 24175e08-b0e0-49a3-b6d3-6a264fe0d9a4
date: 2025-10-17 00:00:40
auther: squaly
featured_image: Pasted%20image%2020261024074637.png
featured_image_preview: Pasted%20image%2020261024074637.png
summary: openwrt经常有版本更新，包括修复漏洞的小更新或功能升级的大更新或新增同步源等。由于x86或虚拟机环境特殊，往往需要扩容，直接升级后有机会丢失部分插件或配置。于是做了一次备份，重装升级的测试。 🧩 一、确认你当前系统可以备份哪些内容 OpenWrt（包括 ImmortalWrt）支持两种备份方
permalink: /archives/LkELh0fI
categories:
  - network
tags:
  - openwrt
---

openwrt经常有版本更新，包括修复漏洞的小更新或功能升级的大更新或新增同步源等。由于x86或虚拟机环境特殊，往往需要扩容，直接升级后有机会丢失部分插件或配置。于是做了一次备份，重装升级的测试。


## 🧩 一、确认你当前系统可以备份哪些内容

OpenWrt（包括 ImmortalWrt）支持两种备份方式：

| 类型                         | 说明                                                                                          |
| ---------------------------- | --------------------------------------------------------------------------------------------- |
| **系统配置备份**       | 保存 `/etc/config/`下的所有系统设置，如网络、防火墙、DHCP、用户、计划任务、插件配置文件等。 |
| **插件与软件列表备份** | 记录你安装的所有软件包（`opkg`包列表），方便恢复后重新安装。                                |

---

## ✅ 二、完整备份步骤

### 1️⃣ 备份配置（最重要）

```bash
sysupgrade -b /tmp/backup.tar.gz
```

生成 `/tmp/backup.tar.gz` 文件。

你可以把它下载到电脑（在 LuCI 网页 → 系统 → 备份/升级 → 下载备份）。

也可以用命令：

```bash
scp root@你的IP:/tmp/backup.tar.gz .
```

---

### 2️⃣ 备份插件列表（软件包）

执行：

```bash
opkg list-installed | awk '{print $1}' > /tmp/pkglist.txt
```

然后下载 `/tmp/pkglist.txt`。

---

## 🧱 三、重新安装（新磁盘/更大空间）

1️⃣ 在宿主机（Proxmox/VMware/VirtualBox）中：

* 删除旧虚拟磁盘；
* 新建一个 **4GB 或 8GB 的虚拟硬盘**；
* 安装最新版本的 ImmortalWrt / OpenWrt。

2️⃣ 启动新系统，确认 `/overlay` 空间变大（例如几 GB）。

---

## ♻️ 四、恢复备份

### 1️⃣ 上传备份文件

将 `backup.tar.gz` 上传到 `/tmp`：

```bash
scp backup.tar.gz root@你的新IP:/tmp/
```

---

### 2️⃣ 恢复配置

```bash
sysupgrade -r /tmp/backup.tar.gz
```

或者通过 LuCI：

* 系统 → 备份/升级 → 恢复备份 → 上传并恢复。

系统会自动重启并恢复所有配置。

---

### 3️⃣ 恢复插件

恢复后执行：

```bash
opkg update
xargs -a /tmp/pkglist.txt opkg install
```

这会自动安装你之前的所有插件（前提是同一架构版本）。

---

### 4️⃣ 检查验证

```bash
df -h
opkg list-installed
```

确认 overlay 空间变大、插件恢复完成、配置一致。

---

## ⚙️ 五、如果版本不同（例如 ImmortalWrt → OpenWrt）

* 大多数配置仍然兼容；
* 如果插件包名不同，恢复时某些包会提示无法找到（可忽略）；
* 恢复后建议手动检查 `/etc/config/network` 和 `/etc/config/firewall`。

---

## 🚀 建议组合命令（一次备份打包）

```bash
sysupgrade -b /tmp/backup.tar.gz
opkg list-installed | awk '{print $1}' > /tmp/pkglist.txt
tar czvf /tmp/full-backup.tgz /tmp/backup.tar.gz /tmp/pkglist.txt
```

下载 `/tmp/full-backup.tgz`，一份搞定所有配置和插件清单。


在 OpenWrt / ImmortalWrt 中，
**系统设置** 和 **插件安装包**、**插件数据** 其实分散在不同地方。

---

## 🧩 一、备份/恢复的范围说明

| 类型                                     | 是否会被 `/etc/config`备份恢复 | 说明                                                   |
| ---------------------------------------- | -------------------------------- | ------------------------------------------------------ |
| 系统设置（网络、DHCP、防火墙、主机名等） | ✅ 会恢复                        | 这些都在 `/etc/config/*`                             |
| 插件安装列表（已安装软件包）             | 🚫 不会自动恢复                  | 需要你保存 `/tmp/pkglist.txt`，重装后批量安装        |
| 插件配置文件（如 openclash 配置）        | ✅ 通常会恢复                    | 只要它的配置在 `/etc/`下（比如 `/etc/openclash/`） |
| 插件运行数据（缓存、日志、临时文件）     | 🚫 不会恢复                      | 一般在 `/tmp`或 `/usr/share`，重启会丢失           |
| Docker、容器、媒体文件等大数据           | 🚫 不会恢复                      | 这些在 `/mnt`、`/overlay`、外接盘，需要手动备份    |

---

## 🧾 二、建议的完整备份方式（可100%还原）

在重新安装前，建议你做 **这三步**：

---

### ✅ **1. 备份系统配置**

```bash
sysupgrade -b /tmp/config_backup.tar.gz
```

然后下载这个文件：

```
/tmp/config_backup.tar.gz
```

这个文件包含 `/etc/config`、`/etc/openclash/` 等配置。

---

### ✅ **2. 保存插件安装列表**

```bash
opkg list-installed | awk '{print $1}' > /tmp/pkglist.txt
```

然后下载：

```
/tmp/pkglist.txt
```

---

### ✅ **3. 备份自定义目录（可选，但建议）**

例如：

```bash
tar -czf /tmp/custom_backup.tar.gz /root /etc/openclash /etc/crontabs
```

这样能保留 OpenClash 的配置文件、订阅地址、自定义脚本等。

---

## ♻️ **恢复步骤（重新安装后）**

1. 上传 `config_backup.tar.gz` 到 `/tmp`
2. 恢复配置：
   ```bash
   sysupgrade -r /tmp/config_backup.tar.gz
   ```
3. 重新安装插件：
   ```bash
   opkg update
   xargs -a /tmp/pkglist.txt opkg install
   ```
4. 如有需要，再解压你手动打包的 `custom_backup.tar.gz`：
   ```bash
   tar -xzf /tmp/custom_backup.tar.gz -C /
   ```

---

## 🧠 结果

恢复完成后：

✅ 所有插件会自动安装回来
✅ 所有系统设置（如网络、OpenClash、DDNS等）会恢复
✅ OpenClash 的订阅、策略组、配置文件也会恢复
🚫 唯一要注意的是：**如果你换了版本太旧/太新的 OpenClash，配置可能不兼容**


**OpenWrt/ImmortalWrt 一键备份脚本**，
执行后会自动生成一个压缩包 `/tmp/full_backup.tar.gz`，
里面包含：

* 系统配置
* 插件列表
* OpenClash 配置
* `/root` 和计划任务等自定义内容

---

## 🧩 **使用方法**

### ① 新建脚本文件：

```bash
cat > /root/full_backup.sh <<'EOF'
#!/bin/sh
set -e

echo "📦 正在生成系统备份，请稍候..."

# 时间戳
DATE=$(date +%Y%m%d-%H%M)

# 路径
BACKUP_DIR="/tmp/openwrt_backup_$DATE"
mkdir -p "$BACKUP_DIR"

# 1️⃣ 系统配置备份
sysupgrade -b "$BACKUP_DIR/config_backup.tar.gz"

# 2️⃣ 插件安装列表
opkg list-installed | awk '{print $1}' > "$BACKUP_DIR/pkglist.txt"

# 3️⃣ 备份重要目录（根据你实际情况可加）
tar -czf "$BACKUP_DIR/custom_data.tar.gz" /root /etc/openclash /etc/crontabs 2>/dev/null || true

# 4️⃣ 打包所有备份为一个文件
cd /tmp
tar -czf "/tmp/full_backup.tar.gz" "openwrt_backup_$DATE"

echo "✅ 备份完成！文件位置：/tmp/full_backup.tar.gz"
echo "📥 请尽快下载并保存此文件。"
EOF
```

---

### ② 赋予执行权限：

```bash
chmod +x /root/full_backup.sh
```

---

### ③ 运行脚本：

```bash
/root/full_backup.sh
```

执行完成后，系统会输出：

```
✅ 备份完成！文件位置：/tmp/full_backup.tar.gz
📥 请尽快下载并保存此文件。
```

---

### ④ 下载备份文件

在浏览器访问：

```
http://你的路由器IP/cgi-bin/luci/admin/system/file_transfer
```

或者使用 WinSCP / scp 下载：

```
/tmp/full_backup.tar.gz
```


## 🧩 **恢复脚本**

### ① 新建脚本文件：

```bash
cat > /root/full_restore.sh <<'EOF'
#!/bin/sh
set -e

BACKUP_FILE="/tmp/full_backup.tar.gz"

if [ ! -f "$BACKUP_FILE" ]; then
    echo "❌ 备份文件 $BACKUP_FILE 不存在，请先上传！"
    exit 1
fi

echo "📦 正在解压备份..."
cd /tmp
tar -xzf "$BACKUP_FILE"

# 找出备份目录
BACKUP_DIR=$(ls -d /tmp/openwrt_backup_* | sort | tail -n1)
if [ ! -d "$BACKUP_DIR" ]; then
    echo "❌ 找不到解压后的备份目录"
    exit 1
fi

echo "✅ 备份目录找到：$BACKUP_DIR"

# 1️⃣ 恢复系统配置
if [ -f "$BACKUP_DIR/config_backup.tar.gz" ]; then
    echo "🔹 正在恢复系统配置..."
    sysupgrade -r "$BACKUP_DIR/config_backup.tar.gz"
else
    echo "⚠️ 未找到系统配置备份"
fi

# 2️⃣ 恢复插件安装列表
if [ -f "$BACKUP_DIR/pkglist.txt" ]; then
    echo "🔹 正在恢复插件..."
    opkg update
    xargs -a "$BACKUP_DIR/pkglist.txt" opkg install || true
else
    echo "⚠️ 未找到插件列表"
fi

# 3️⃣ 恢复自定义数据（OpenClash / root / crontabs）
if [ -f "$BACKUP_DIR/custom_data.tar.gz" ]; then
    echo "🔹 正在恢复自定义数据..."
    tar -xzf "$BACKUP_DIR/custom_data.tar.gz" -C /
else
    echo "⚠️ 未找到自定义数据"
fi

echo "✅ 恢复完成！请重启系统以确保所有服务正常运行。"
EOF
```

---

### ② 赋予执行权限：

```bash
chmod +x /root/full_restore.sh
```

---

### ③ 上传备份文件

将之前下载的 `full_backup.tar.gz` 上传到：

```
/tmp/full_backup.tar.gz
```

可以用 `scp` 或 WinSCP 上传。

---

### ④ 运行恢复脚本：

```bash
/root/full_restore.sh
```

脚本会自动完成：

1. 恢复系统配置（包括网络、防火墙、OpenClash 配置等）
2. 重新安装你之前的所有插件
3. 恢复自定义目录 `/root`、`/etc/openclash`、`/etc/crontabs` 等

---

### ⑤ 完成后重启系统：

```bash
reboot
```

✅ 系统、插件、OpenClash 配置都会恢复到重装前的状态。
