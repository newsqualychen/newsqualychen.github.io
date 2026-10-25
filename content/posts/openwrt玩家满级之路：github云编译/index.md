---
title: openwrt玩家满级之路：github云编译
id: 019d22d1-2438-71de-8dd5-cd09b386ee82
date: 2026-03-25 10:27:09
auther: squaly
featured_image: Pasted%20image%2020261024074601.png
featured_image_preview: Pasted%20image%2020261024074601.png
summary: 为什么最终要自己编译openwrt固件？不能直接下载官方img？ 官方的img容量太低只有几百m，完全不够用，安装好后好要扩容（参考我之前的文章），这本身也不是什么问题，但是openwrt版本升级太快，扩容后升级会清空所有安装的软件和设置，每次升级重新设置一次显得非常痛苦，开源的社区默认每个人都有动
permalink: /archives/oBQDu1lL
categories:
  - network
tags:
  - openwrt
---

### 为什么最终要自己编译openwrt固件？不能直接下载官方img？

官方的img容量太低只有几百m，完全不够用，安装好后好要扩容（参考我之前的文章），这本身也不是什么问题，但是openwrt版本升级太快，扩容后升级会清空所有安装的软件和设置，每次升级重新设置一次显得非常痛苦，开源的社区默认每个人都有动手修改的能力，最终需要走到自己编译这部。

### 为什么不本地编译？

本地编译需要非常好的的网络环境，需要不断拉取更新数据源，如果节点有延时，很容易会下载失败，这时系统会自动跳过下载直接执行下一步，无论怎样编译经常会失败。出错后还需要排查。**如果能解决网络问题，优先本地编译，速度快很多。**

推荐本地编译使用的配置：ubuntu 8核的cpu，8g内存，200g硬盘（越大越好）我在另外的文章单独讲本地编译。

## 教程开始

fork仓库：[https://github.com/P3TERX/Actions-OpenWrt](https://github.com/P3TERX/Actions-OpenWrt)

或者fork我的仓库修改：[https://github.com/newsqualychen/openwrt-x86-64](https://github.com/newsqualychen/openwrt-x86-64)

## 使用方法：

### 1. 修改.config文件：

```
# ==================== 架构与分区 ====================
CONFIG_TARGET_x86=y
CONFIG_TARGET_x86_64=y
CONFIG_TARGET_x86_64_DEVICE_generic=y

# 分区大小（建议 12GB，够用且不浪费）
CONFIG_TARGET_KERNEL_PARTSIZE=64
CONFIG_TARGET_ROOTFS_PARTSIZE=12288

# ==================== 万兆网卡驱动 ====================
CONFIG_PACKAGE_kmod-mlx5-core=y
CONFIG_PACKAGE_kmod-mlx5-lib=y

# ==================== 核心加速与网络 ====================
CONFIG_PACKAGE_dnsmasq-full=y
CONFIG_PACKAGE_ip-full=y
CONFIG_PACKAGE_kmod-nft-offload=y          # 软件流卸载，保留即可

# ==================== 实用插件 ====================
CONFIG_PACKAGE_luci-app-diskman=y
CONFIG_PACKAGE_luci-app-lucky=y
CONFIG_PACKAGE_tailscale=y
CONFIG_PACKAGE_luci-app-ddns-go=y
CONFIG_PACKAGE_luci-app-openclash=y        # 推荐主力科学上网
# CONFIG_PACKAGE_luci-app-passwall=y       # 如需可取消注释，但建议先只用 OpenClash
CONFIG_PACKAGE_luci-app-nlbwmon=y
CONFIG_PACKAGE_luci-app-wol=y
CONFIG_PACKAGE_luci-app-filemanager=y
CONFIG_PACKAGE_luci-app-appfilter=y
CONFIG_PACKAGE_luci-app-upnp=y
CONFIG_PACKAGE_luci-app-unblockneteasemusic=y
CONFIG_PACKAGE_luci-app-qos=y
CONFIG_PACKAGE_luci-app-usb-printer=y
CONFIG_PACKAGE_luci-i18n-usb-printer-zh-cn=y
CONFIG_PACKAGE_luci-app-vlmcsd=y

# ==================== 主题与中文 ====================
CONFIG_PACKAGE_luci-theme-argon=y
CONFIG_PACKAGE_luci-theme-material=y
CONFIG_PACKAGE_luci-i18n-base-zh-cn=y

# ==================== 工具 ====================
CONFIG_PACKAGE_fdisk=y
CONFIG_PACKAGE_resize2fs=y

# 可选增强（根据需要取消注释）
# CONFIG_PACKAGE_luci-app-samba4=y
# CONFIG_PACKAGE_luci-app-alist=y
# CONFIG_PACKAGE_docker=y
# CONFIG_PACKAGE_dockerd=y
# CONFIG_PACKAGE_luci-app-docker=y
# CONFIG_PACKAGE_kmod-fs-exfat=y
# CONFIG_PACKAGE_kmod-fs-ntfs3=y
```

### 2.修改脚本：diy-part2.sh

**提示**：你可以去 [ASCII Generator](http://www.network-science.de/ascii/) 网站生成你喜欢的字体样式，然后把生成的字符粘贴到上面的引号中间。

可选：把SqualyWrt换成你自己喜欢的名字：例如：mywrt

把ip地址换成自己的地址

```
#!/bin/bash
#
# https://github.com/P3TERX/Actions-OpenWrt
# File name: diy-part2.sh
# Description: OpenWrt DIY script part 2 (After Update feeds)
#
# Copyright (c) 2019-2024 P3TERX <https://p3terx.com>
#
# This is free software, licensed under the MIT License.
# See /LICENSE for more information.

# ====================== 你的基础修改 ======================

# 1. 修改默认 IP 为你自己的网段，例如： 192.168.10.1
sed -i 's/192.168.1.1/192.168.10.1/g' package/base-files/files/bin/config_generate

# 2. 修改默认主机名
sed -i 's/ImmortalWrt/SqualyWrt/g' package/base-files/files/bin/config_generate

# 3. 修改 Banner（SSH 登录时的字符画 Logo）
cat > package/base-files/files/etc/banner << 'EOF'
 ____                              ___                               __  
/\  _`\                           /\_ \                             /\ \__   
\ \,\L\_\     __   __  __     __  \//\ \    __  __  __  __  __  _ __\ \ ,_\  
 \/_\__ \   /'__`\/\ \/\ \  /'__`\  \ \ \  /\ \/\ \/\ \/\ \/\ \/\`'__\ \ \/  
   /\ \L\ \/\ \L\ \ \ \_\ \/\ \L\.\_ \_\ \_\ \ \_\ \ \ \_/ \_/ \ \ \/ \ \ \_ 
   \ `\____\ \___, \ \____/\ \__/.\_\/\____\\/`____ \ \___x___/'\ \_\  \ \__\
    \/_____/\/___/\ \/___/  \/__/\/_/\/____/ `/___/> \/__//__/   \/_/   \/__/
                 \ \_\                          /\___/   
                  \/_/                          \/__/  

 --- Welcome to SqualyWrt | 万兆版 | ImmortalWrt Base ---
EOF

# 4. 修改版本号后缀
sed -i "s/OpenWrt /Built by SqualyWrt /g" package/base-files/files/etc/openwrt_release

# ====================== 默认主题（Bootstrap）Web UI 修改 ======================

# 修改网页标题、顶部 Logo 文字和页脚
echo "=== 开始修改默认 Bootstrap 主题的 Web UI Logo 和页脚 ==="

# 替换 header 中的品牌名称
find feeds/luci -name "header.htm" -path "*/themes/bootstrap/*" -exec sed -i 's/ImmortalWrt/SqualyWrt/g' {} + 2>/dev/null || true
find feeds/luci -name "header.htm" -path "*/themes/bootstrap/*" -exec sed -i 's/OpenWrt/SqualyWrt/g' {} + 2>/dev/null || true

# 替换 footer 中的 Powered by 文字
find feeds/luci -name "footer.htm" -path "*/themes/bootstrap/*" -exec sed -i 's/Powered by .*OpenWrt.*/Powered by SqualyWrt (基于 ImmortalWrt)/g' {} + 2>/dev/null || true
find feeds/luci -name "footer.htm" -path "*/themes/bootstrap/*" -exec sed -i 's/ImmortalWrt/SqualyWrt/g' {} + 2>/dev/null || true

# 额外修改 LuCI 基础模块中的标题（防止某些页面仍显示原名称）
sed -i 's/ImmortalWrt/SqualyWrt/g' feeds/luci/modules/luci-base/luasrc/controller/admin/index.lua 2>/dev/null || true
sed -i 's/OpenWrt/SqualyWrt/g' feeds/luci/modules/luci-base/luasrc/controller/admin/index.lua 2>/dev/null || true

# 如果你的 config 里还勾选了 luci-theme-material，也可以顺便改一下（保险起见）
find feeds/luci -name "header.htm" -path "*/themes/material/*" -exec sed -i 's/ImmortalWrt/SqualyWrt/g' {} + 2>/dev/null || true
find feeds/luci -name "header.htm" -path "*/themes/material/*" -exec sed -i 's/OpenWrt/SqualyWrt/g' {} + 2>/dev/null || true

echo "=== diy-part2.sh 执行完成：默认 IP、主机名、Banner、Bootstrap 默认主题 Web Logo & Footer 已修改 ==="
```

修改插件源:diy-part1.sh

```
#!/bin/bash
#
# https://github.com/P3TERX/Actions-OpenWrt
# File name: diy-part1.sh
# Description: OpenWrt DIY script part 1 (Before Update feeds)
#
# Copyright (c) 2019-2024 P3TERX <https://p3terx.com>
#
# This is free software, licensed under the MIT License.
# See /LICENSE for more information.
#

# Uncomment a feed source
#sed -i 's/^#\(.*helloworld\)/\1/' feeds.conf.default

# Add a feed source
echo 'src-git helloworld https://github.com/fw876/helloworld' >>feeds.conf.default
#echo 'src-git passwall https://github.com/xiaorouji/openwrt-passwall' >>feeds.conf.default
#!/bin/bash
# 添加 Lucky 插件源
git clone https://github.com/gdy666/luci-app-lucky.git package/lucky
# 添加 DDNS-go 插件源
git clone https://github.com/sirpdboy/luci-app-ddns-go.git package/ddns-go
# 添加 OpenClash (如果官方源没有)
git clone --depth=1 -b master https://github.com/vernesong/OpenClash.git package/openclash
```

### 3. 修改工作流文件：

```
#
# https://github.com/P3TERX/Actions-OpenWrt
#
# File: .github/workflows/openwrt-builder.yml
# Description: Build OpenWrt using GitHub Actions
#

name: ImmortalWrt x86-64 Builder   # ← 修改：让 Actions 页面显示更清晰

on:
  repository_dispatch:
  workflow_dispatch:

env:
  REPO_URL: https://github.com/immortalwrt/immortalwrt
  REPO_BRANCH: master          # ← 如果你想用 24.10 分支，改成 24.10
  FEEDS_CONF: feeds.conf.default
  CONFIG_FILE: .config
  DIY_P1_SH: diy-part1.sh
  DIY_P2_SH: diy-part2.sh
  UPLOAD_BIN_DIR: false
  UPLOAD_FIRMWARE: true
  UPLOAD_RELEASE: true
  TZ: Asia/Shanghai

jobs:
  build:
    # ========== 关键修改1：使用更大规格的 runner（推荐）==========
    # 如果你 GitHub 有 Pro/Team/Enterprise 权限，可以直接用 ubuntu-22.04-16core
    # 免费用户若无法使用 larger runner，请先改回 ubuntu-22.04 并重点靠下面的 Swap
    runs-on: ubuntu-22.04-16core     # ← 修改这里（推荐）
    # runs-on: ubuntu-22.04          # ← 如果 larger runner 不可用，改回这行

    timeout-minutes: 480             # ← 新增：给 8 小时，避免 GitHub 强制中断

    steps:
    - name: Checkout
      uses: actions/checkout@main

    - name: Initialization environment
      env:
        DEBIAN_FRONTEND: noninteractive
      run: |
        sudo rm -rf /etc/apt/sources.list.d/* /usr/share/dotnet /usr/local/lib/android /opt/ghc /opt/hostedtoolcache/CodeQL
        sudo docker image prune --all --force
        sudo -E apt-get -qq update
        sudo -E apt-get -qq install ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential bzip2 ccache cmake cpio curl device-tree-compiler fastjar flex gawk gettext gcc-multilib g++-multilib git gperf haveged help2man intltool libc6-dev-i386 libelf-dev libfuse-dev libglib2.0-dev libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev libncurses5-dev libncursesw5-dev libpython3-dev libreadline-dev libssl-dev libtool lrzsz mkisofs msmtp ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pyelftools python3-setuptools qemu-utils rsync scons squashfs-tools subversion swig texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
        sudo -E apt-get -qq autoremove --purge
        sudo -E apt-get -qq clean
        sudo timedatectl set-timezone "$TZ"
        sudo mkdir -p /workdir
        sudo chown $USER:$GROUPS /workdir

    # ========== 关键修改2：增加 Swap（防 OOM）==========
    - name: Setup Swap (16GB) to prevent OOM
      run: |
        sudo fallocate -l 16G /swapfile
        sudo chmod 600 /swapfile
        sudo mkswap /swapfile
        sudo swapon /swapfile
        echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
        free -h

    # ========== 关键修改3：添加缓存（大幅省时间和内存）==========
    - name: Cache OpenWrt build files
      uses: actions/cache@v4
      with:
        path: |
          /workdir/openwrt/dl
          /workdir/openwrt/ccache
          /workdir/openwrt/staging_dir
          /workdir/openwrt/build_dir
        key: ${{ runner.os }}-immortalwrt-${{ hashFiles('.config') }}
        restore-keys: |
          ${{ runner.os }}-immortalwrt-

    - name: Clone source code
      working-directory: /workdir
      run: |
        df -hT $PWD
        git clone $REPO_URL -b $REPO_BRANCH openwrt
        ln -sf /workdir/openwrt $GITHUB_WORKSPACE/openwrt

    - name: Load custom feeds
      run: |
        [ -e $FEEDS_CONF ] && mv $FEEDS_CONF openwrt/feeds.conf.default
        chmod +x $DIY_P1_SH
        cd openwrt
        $GITHUB_WORKSPACE/$DIY_P1_SH

    - name: Update feeds
      run: cd openwrt && ./scripts/feeds update -a

    - name: Install feeds
      run: cd openwrt && ./scripts/feeds install -a

    - name: Load custom configuration
      run: |
        [ -e files ] && mv files openwrt/files
        [ -e $CONFIG_FILE ] && mv $CONFIG_FILE openwrt/.config
        chmod +x $DIY_P2_SH
        cd openwrt
        $GITHUB_WORKSPACE/$DIY_P2_SH

    - name: Download package
      id: package
      run: |
        cd openwrt
        make defconfig
        make download -j8

    - name: Compile the firmware
      id: compile
      run: |
        cd openwrt
        echo "Using 4~6 threads to avoid OOM"
        make -j4 || make -j2 || make -j1 V=s     # ← 修改：限制线程数，防 OOM
        echo "status=success" >> $GITHUB_OUTPUT
        grep '^CONFIG_TARGET.*DEVICE.*=y' .config | sed -r 's/.*DEVICE_(.*)=y/\1/' > DEVICE_NAME
        [ -s DEVICE_NAME ] && echo "DEVICE_NAME=_$(cat DEVICE_NAME)" >> $GITHUB_ENV
        echo "FILE_DATE=_$(date +"%Y%m%d%H%M")" >> $GITHUB_ENV

    - name: Check space usage
      if: (!cancelled())
      run: df -hT

    # 后面 Upload 部分保持不变（可根据需要微调）
    - name: Upload bin directory
      uses: actions/upload-artifact@main
      if: steps.compile.outputs.status == 'success' && env.UPLOAD_BIN_DIR == 'true'
      with:
        name: OpenWrt_bin${{ env.DEVICE_NAME }}${{ env.FILE_DATE }}
        path: openwrt/bin

    - name: Organize files
      id: organize
      if: env.UPLOAD_FIRMWARE == 'true' && !cancelled()
      run: |
        cd openwrt/bin/targets/*/*
        rm -rf packages
        echo "FIRMWARE=$PWD" >> $GITHUB_ENV
        echo "status=success" >> $GITHUB_OUTPUT

    - name: Upload firmware directory
      uses: actions/upload-artifact@main
      if: steps.organize.outputs.status == 'success' && !cancelled()
      with:
        name: OpenWrt_firmware${{ env.DEVICE_NAME }}${{ env.FILE_DATE }}
        path: ${{ env.FIRMWARE }}

    - name: Generate release tag
      id: tag
      if: env.UPLOAD_RELEASE == 'true' && !cancelled()
      run: |
        echo "release_tag=$(date +"%Y.%m.%d-%H%M")" >> $GITHUB_OUTPUT
        touch release.txt
        echo "status=success" >> $GITHUB_OUTPUT

    - name: Upload firmware to release
      uses: softprops/action-gh-release@master
      if: steps.tag.outputs.status == 'success' && !cancelled()
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        tag_name: ${{ steps.tag.outputs.release_tag }}
        body_path: release.txt
        files: ${{ env.FIRMWARE }}/*

    - name: Delete old workflow runs
      uses: Mattraks/delete-workflow-runs@main
      with:
        retain_days: 0
        keep_minimum_runs: 2

    - name: Remove old Releases
      uses: dev-drprasad/delete-older-releases@master
      if: env.UPLOAD_RELEASE == 'true' && !cancelled()
      with:
        keep_latest: 3
        delete_tags: true
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 4. 添加权限

为了让你以后能在 **Releases** 页面直接看到带日期、可直接下载的固件包，请做如下设置：

1. 进入你 GitHub 仓库的 **Settings** (设置)。
2. 在左侧菜单点击 **Actions** -> **General**。
3. 翻到页面最下面，找到 **Workflow permissions**。
4. 将选项从“Read repository contents...”改为 **"Read and write permissions"**。
5. 勾选下面的 **"Allow GitHub Actions to create and approve pull requests"**。
6. 点击 **Save**。

**触发编译**：点击 GitHub 的 Actions 选项卡，手动触发编译。

**获取固件**：大约 1-4 小时后，你就能下载到专属的 `.img.gz`

### 5. 常见错误：

大部份人加了自己喜欢的插件或主题后普通的github帐号服务器资源不够，就会报错：

```
The hosted runner lost communication with the server. Anything in your workflow that terminates the runner process, starves it for CPU/Memory, or blocks its network access can cause this error.
```

**GitHub Actions 机器被干掉了（不是编译错误）**

具体就是：

💥 内存 / IO / 磁盘 / CPU 被榨干 → runner 被系统强制回收

**常见触发点（大概率踩了其中1-2个）**

#### 1.编译中途内存峰值（最常见）

比如：

* gcc / ld 链接阶段
* 编译 python / node / llvm

一瞬间吃掉：

<pre class="overflow-visible! px-0!" data-start="637" data-end="667"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>6GB → 9GB → 💥 OOM</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

#### 2. IO 打满（你用了缓存的话更容易）

<pre class="overflow-visible! px-0!" data-start="698" data-end="746"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>restore cache → 解压 → IO爆炸 → runner卡死</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

### 3. 磁盘不够（隐藏杀手）

GitHub runner：

<pre class="overflow-visible! px-0!" data-start="786" data-end="808"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>总空间 ≈ 14GB</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

但 OpenWrt：

<pre class="overflow-visible! px-0!" data-start="822" data-end="865"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>build_dir + staging_dir ≈ 20GB+</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

结果：

<pre class="overflow-visible! px-0!" data-start="874" data-end="911"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>写一半 → 磁盘满 → runner异常 → 掉线</span></div></div></div></div></div></div></div></div></div></div></div></div></pre>

### 6. 解决方案：

#### 方案1（最推荐）：直接本地编译

**这是唯一“稳定”的方案**

本地编译 = 完全碾压 GitHub Actions

#### 方案2：继续用 Actions（但要“极限降载”）

如果你必须用 Actions，必须这样改👇

#### 1. 强制单线程（关键）

<pre class="overflow-visible! px-0!" data-start="1155" data-end="1179"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼd">make</span><span> </span><span class="ͼf">-j1</span><span> </span><span class="ͼe">V</span><span class="ͼ8">=</span><span>s</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

👉 不要犹豫，直接 `-j1`

#### 2. 彻底精简 `.config`（重点）

必须删：

* onionshare-cli（你已经有 warning）
* python相关
* docker
* luci多主题
* 不用的驱动

---

#### 3. 禁用大缓存（很多人忽略）

只保留：

<pre class="overflow-visible! px-0!" data-start="1353" data-end="1374"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>dl</span><br/><span>ccache</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

禁止：

<pre class="overflow-visible! px-0!" data-start="1382" data-end="1415"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>build_dir</span><br/><span>staging_dir</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

---

#### 4. 限制 download

<pre class="overflow-visible! px-0!" data-start="1444" data-end="1473"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼd">make</span><span> download </span><span class="ͼf">-j2</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

---

#### 5. swap 保持 8G（不要16G）

---

### 如何判断是不是资源问题（教你看日志）

下次失败时看：

#### 如果是这个结尾：

<pre class="overflow-visible! px-0!" data-start="1568" data-end="1586"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>Killed</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

= 内存炸了（100%）

#### 如果是：

<pre class="overflow-visible! px-0!" data-start="1622" data-end="1657"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>No space left on device</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

= 磁盘炸了

#### 如果是：

<pre class="overflow-visible! px-0!" data-start="1687" data-end="1717"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute end-1.5 top-1 z-2 md:end-2 md:top-1"></div><div class="pe-11 pt-3"><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>lost communication</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

= IO / CPU / OOM 任一触发

**恭喜看完这遍文章，如果你走上了\*\*“GitHub 自建编译”\*\*这条路，你其实已经开启了 OpenWrt 玩家的“满级模式”。**

## Q&A：

### 1. 软件本身会保留吗？

**会保留，但有前提条件。**

当你在 OpenWrt 后台上传新固件并点击“执行脚本升级”时，系统会默认勾选 **“保留当前配置”**。

* **系统设置**：你的 LAN 口 IP (10.10.10.252)、防火墙规则、主机名、拨号信息等都会被完整保留。
* **软件设置**：Lucky、DDNS-go、OpenClash 的规则和账号配置，只要它们的配置文件路径在 `/etc/config/` 或系统默认保留列表里，都会自动迁移到新系统。

**这就是自己编译的巨大优势：**

* **如果你在网页版安装**：升级后，通过 `opkg install` 手动安装的软件通常会**消失**，你需要重新安装一遍。
* **如果你在 GitHub 编译里集成了软件**：由于 Lucky、DDNS-go 已经“焊死”在你的固件里了，升级完重启，它们**依然在那里**，且配置也是旧的，直接就能用。

### 2. 以后升级的流程

以后你只需要：

1. 修改 `diy-part1.sh`（如果有新插件想加）。
2. 点击 **Run workflow**。
3. 等待完成后，在仓库的 **Releases** 处下载新固件。
4. 进 OpenWrt 后台上传升级。

**由于你已经把设置都写进了脚本和配置里，升级完后，你的 IP、驱动、插件和 12GB 空间都会自动保持，真正的“值守式升级”。**

### 3. 应该用哪个文件？

在 ESXi 虚拟机环境下，请毫不犹豫地选择：**`openwrt-x86-64-generic-squashfs-combined-efi.img.gz`**

* **为什么选它：**
  * **Combined**：表示包含引导分区（Kernel）和根分区（Rootfs），是一个完整的磁盘镜像。
  * **EFI**：你的 ESXi 虚拟机如果设置的是 EFI 引导（现代虚拟机的默认推荐），必须用这个。
  * **.img.gz**：这是压缩包。解压后得到 `.img` 文件，你可以直接用工具（如 StarWind V2V）转成 ESXi 的 `.vmdk`，或者直接在 ESXi 8.0 中作为硬盘映像导入。

### 4. 关于 ext4 和 squashfs 的纠结

固件文件名里带有 **`squashfs`**，这确实和你想用的 **`ext4`** 不同。

### 5.为什么编译产物默认是 squashfs？

* **自带“后悔药”**：Squashfs 是一种只读文件系统。它的精妙之处在于，所有的修改都存在另一个层（overlay）上。如果你哪天把系统配置玩挂了，直接在后台点“恢复出厂设置”，它能秒回初始状态。
* **稳定性**：在断电或异常关机时，只读分区不会损坏。

#### 5.1 ext4 的优势

* **读写更自由**：它不像 squashfs 那样分层，修改直接写入磁盘。
* **易于扩容**：在 ESXi 里拉伸磁盘空间时，ext4 的操作确实更直观。

---

#### 5.2. 我该重新编译 ext4 吗？

**如果你非常看重磁盘性能和极致的扩容便利性，可以考虑重新编译。**

在 GitHub 编译的 `.config` 文件里，你可以找到这一行：`# CONFIG_TARGET_ROOTFS_EXT4FS is not set`将其修改为：`CONFIG_TARGET_ROOTFS_EXT4FS=y`

**但是，我给你的建议是：先用这个 squashfs。**因为你在编译时已经把分区大小定死在 **2GB** 了，对于 OpenWrt 来说，2GB 的空间即便是用 squashfs 也是绰绰有余的（大部分插件只占用几百 MB）。你根本不需要再经历手动扩容的痛苦。

### 6. 落地 ESXi 的最终步骤

1. **解压**：得到 `...combined-efi.img`。
2. **转换**：不要用解压工具，多数报错，直接在终端cd到该文件夹
   ```
   gunzip 你的固件名称.img.gz
   ```
3. windows使用 **StarWind V2V Converter** 将其转为 `VMware ESXi pre-allocated image`。
4. **新建虚拟机**：
   * **重要**：StarWind V2V Converter上传硬盘后修改容量为你编译的容量。
   * **网卡**：别忘了勾选你之前集成的 Mellanox 万兆网卡直通，或者使用 VMXNET3。
   * **引导**：确保虚拟机选项里的“启动选项”设为 **EFI**。（取消勾选uefi）
