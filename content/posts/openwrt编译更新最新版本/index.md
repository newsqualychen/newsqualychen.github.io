---
title: openwrt编译更新最新版本
id: 019f0d8a-c0da-7776-b97c-e5ec115ab681
date: 2026-06-28 17:23:38
auther: squaly
featured_image: Pasted%20image%2020261024073853.png
featured_image_preview: Pasted%20image%2020261024073853.png
summary: 请尝试执行以下命令来刷新远程数据：
permalink: /archives/0RzL2Si4
categories:
  - network
tags:
  - openwrt
---

请尝试执行以下命令来刷新远程数据：

```
# 1. 抓取远程仓库的所有最新分支和标签
git fetch --tags origin

# 2. 再次查看最后20个标签，看看有没有新版本出现
git tag | tail -20
```

### 3. 如何切换到你想编译的版本？

你当前处于 **“头指针分离” (Detached HEAD)** 状态，这意味着你直接切换到了一个具体的标签（`v25.12.0-rc2`），而不是在动态更新的分支上。

* **如果你想切换到最新的稳定分支（例如 24.10 分支，让它以后能随时 `git pull` 更新）：**
* **如果你想锁死在某个具体的正式版标签（例如 `v24.10.5`）：**

  ```
  git checkout v24.10.5
  ```

  *(注意：切换 tag 之后就会进入你当前的 Detached HEAD 状态，这是正常的，适合用来做单次固定版本的编译。)*

如果修改过

`feeds.conf.default` 文件（通常是为了添加第三方插件源），而 Git 为了保护你的修改不被冲掉，拒绝了直接切换。

别慌，处理这种冲突通常有以下两种最简单的方法，取决于你想不想**保留**对这个文件的修改：

### 方法 A：放弃本地修改（最省事，推荐）

如果你不记得自己改了啥，或者只是想用纯净的官方 `v25.12.0` 源码，直接强制覆盖它：

```
# 强制让这个文件恢复到当前版本的初始状态
git checkout -- feeds.conf.default

# 然后再次切换到新版本
git checkout v25.12.0
```

### 方法 B：暂存修改，切过去后再恢复（保留你的配置）

如果你之前在 `feeds.conf.default` 里加了自定义的插件源（比如某些特殊的软件仓库），并且想在新版本里继续用：

```
# 1. 把修改临时存进 Git 的“小仓库”（暂存区）
git stash

# 2. 这时由于文件恢复干净了，可以顺利切换版本
git checkout v25.12.0

# 3. 把刚才存起来的修改弹出来，应用到新版本上
git stash pop
```

> ⚠️ **注意：** 执行 `git stash pop` 时，如果新版本的 `feeds.conf.default` 官方也改了同一处地方，可能会提示 `CONFLICT (content): Merge conflict...`（代码冲突）。 如果遇到冲突，最简单的办法是直接用文本编辑器（如 `nano feeds.conf.default`）打开它，手动删掉 Git 标记的冲突符号（`<<<<<<<`, `=======`, `>>>>>>>`），只留下你想要的那几行。

处理完这一步后，就可以继续后面的 `./scripts/feeds update -a` 流程了！

### 1. 更新并安装插件源 (Feeds)

因为版本从 `-rc2` 升级到了正式版，插件源必须同步更新，否则编译时会因为依赖不匹配而报错：

```
./scripts/feeds update -a && ./scripts/feeds install -a
```

拉取源码过程中有可能会因为网络问题出错,需要保持网络良好.

如果出错执行以下命令,没出错的直接无视

```
# 1. 清理可能下载了一半的错误缓存
rm -rf ./feeds ./package/feeds

# 2. 重新跑更新和安装
./scripts/feeds update -a && ./scripts/feeds install -a
```

#### 验证警告信息

执行完上面的强装命令后，看一眼终端输出。

* **安全的警告（可以继续）：** 如果剩下的警告只是类似 `ruby/host`, `libpam`, `iperf3` 等不属于你必选的第三方小插件，这就**完全正常**，可以忽略。
* **必须消失的警告：** 只要关于 **`luci`** 和 **`luci-i18n-base-zh-cn`** 的那两行警告消失了，就说明网页后台已经就位！

### 2. 刷新配置文件

用官方新版本的默认规则来规范你现有的 `.config` 配置文件：

```
make defconfig
```

> ⚙️ **提示：** 如果你想进入菜单调整或者添加新的固件插件，可以在这一步执行 `make menuconfig`。如果不需要修改，直接跳到下一步。

### 3. 开始多线程编译

直接开满你的 CPU 核心全力输出：

```
make -j$(nproc) V=s
```

2006-08-04更新

新版本使用apk更新,如果运行后发现更新软件包失败,执行以下命令

官方源

```
sed -i 's|https://mirrors.vsean.net/openwrt|https://downloads.immortalwrt.org|g' /etc/apk/repositories.d/distfeeds.list
apk update
```

还有两个报错,可以选择无视,强逼症请继续修复.

```
ERROR: wget: exited with error 8
WARNING: updating and opening https://downloads.immortalwrt.org/releases/25.12.1/packages/x86_64/lucky/packages.adb: unexpected end of file
ERROR: wget: exited with error 8
WARNING: updating and opening https://downloads.immortalwrt.org/releases/25.12.1/packages/x86_64/video/packages.adb: unexpected end of file
 [https://downloads.immortalwrt.org/releases/25.12.1/targets/x86/64/packages/packages.adb]
 [https://downloads.immortalwrt.org/releases/25.12.1/packages/x86_64/base/packages.adb]
 [https://downloads.immortalwrt.org/releases/25.12.1/packages/x86_64/luci/packages.adb]
 [https://downloads.immortalwrt.org/releases/25.12.1/packages/x86_64/packages/packages.adb]
 [https://downloads.immortalwrt.org/releases/25.12.1/packages/x86_64/routing/packages.adb]
 [https://downloads.immortalwrt.org/releases/25.12.1/packages/x86_64/telephony/packages.adb]
2 unavailable, 0 stale; 10462 distinct packages available
```

提示lucky和video更新失败,这两个源对大多数人不是必需的，可以暂时忽略，或者直接删掉它们。

### 推荐操作

**方法 1：直接忽略（最简单）**

现在已经可以正常安装软件了，例如：

```
apk update
apk add luci-app-xxx   # 你需要的软件
```

**方法 2：删除有问题的源（干净）**

```
# 查看当前源
cat /etc/apk/repositories.d/distfeeds.list

# 编辑删除 lucky 和 video 这两行
vi /etc/apk/repositories.d/distfeeds.list
# 或者用 sed 自动删除
sed -i '/lucky/d' /etc/apk/repositories.d/distfeeds.list
sed -i '/video/d' /etc/apk/repositories.d/distfeeds.list

# 再更新一次
apk update
```

### 推荐操作

```
# 安装完整 wget
apk add wget-ssl

# 让 wget 优先使用 IPv4（只影响下载软件包）
echo 'prefer-family = IPv4' >> /etc/wgetrc

# 测试
apk update
```

执行完后再去 WebUI 点更新列表，一般就能正常了。

如果以后想改回，删除 /etc/wgetrc 里那一行即可。

编译前一次性修复，让以后刷出来的固件默认就正常。

在源码目录执行以下命令：

```
cd /immortalwrt
```

然后完整执行下面这一段：

```
cd /home/squaly/openwrt/immortalwrt

mkdir -p files/etc/uci-defaults

cat > files/etc/uci-defaults/99-fix-apk << 'EOF'
#!/bin/sh

# 设置软件源（官方源）
MIRROR="https://downloads.immortalwrt.org"

uci -q get system.@imm_init[0] >/dev/null || uci add system imm_init >/dev/null
uci set system.@imm_init[0].apk_mirror="$MIRROR"
uci commit system

# 替换软件源文件
if [ -f /etc/apk/repositories.d/distfeeds.list ]; then
    sed -i "s|https://mirrors.vsean.net/openwrt|$MIRROR|g" /etc/apk/repositories.d/distfeeds.list
    sed -i "s|https://downloads.immortalwrt.org|$MIRROR|g" /etc/apk/repositories.d/distfeeds.list
fi

# 让 wget 优先使用 IPv4（解决 WebUI 更新失败）
echo 'prefer-family = IPv4' > /etc/wgetrc

exit 0
EOF
```

3. 执行完后检查文件是否创建成功：

```
cat files/etc/uci-defaults/99-fix-apk
chmod +x files/etc/uci-defaults/99-fix-apk
```

看到完整内容后，就可以继续编译了：

### 额外建议（可选但推荐）

编译时在 make menuconfig 里确保选中：

* Base system → wget-ssl（勾选）
* LuCI → Collections → luci（已选的话保持）

然后正常编译：
