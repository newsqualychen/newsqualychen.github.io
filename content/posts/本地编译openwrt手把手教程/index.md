---
title: 本地编译openwrt手把手教程
id: 019d282f-73ce-74dd-b8b1-1249c70fd507
date: 2026-03-26 11:28:20
auther: squaly
featured_image: Pasted%20image%2020261024074353.png
featured_image_preview: Pasted%20image%2020261024074353.png
summary: 有很多人用github云编译的时候遇到错误，如果是fork我的仓库基本都会成功，但是加了自己喜欢的插件或主题后普通的github帐号服务器资源不够，就会反复报错，如果实在没耐心慢慢测试配置，可以在本地编译，下面说说配置： 系统：ubuntu cpu：8核以上 内存：8g（推荐16g） 硬盘：200g
permalink: /archives/I4m4wvzd
categories:
  - network
tags:
  - openwrt
---

**有很多人用github云编译的时候遇到错误，如果是fork我的仓库基本都会成功，但是加了自己喜欢的插件或主题后普通的github帐号服务器资源不够，就会反复报错，如果实在没耐心慢慢测试配置，可以在本地编译，下面说说配置：**

系统：ubuntu

cpu：8核以上

内存：8g（推荐16g）

硬盘：200g以上（推荐ssd）

安装ubuntu和配置系统环境之前的文章有写，不再重复了。

ssh进系统后执行以下命令

新建文件：`openwrt-env-install.sh`

```
#!/usr/bin/env bash

set -e

echo "=== OpenWrt 编译环境初始化开始 ==="

# 更新系统
sudo apt update -y
sudo apt upgrade -y

# 安装基础依赖
sudo apt install -y \
build-essential clang flex bison g++ gawk gcc-multilib g++-multilib \
gettext git libncurses5-dev libssl-dev python3-distutils rsync unzip zlib1g-dev \
file wget python3 python3-pip python3-setuptools \
libelf-dev libglib2.0-dev libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev \
libncursesw5-dev libreadline-dev libsqlite3-dev libtool \
lzop patch pkgconf sudo swig texinfo time xsltproc \
ccache curl subversion

# 可选但推荐
sudo apt install -y \
vim htop tree jq

# 创建工作目录
mkdir -p ~/openwrt-build
cd ~/openwrt-build

# 设置ccache（加速编译）
if ! grep -q ccache ~/.bashrc; then
  echo 'export USE_CCACHE=1' >> ~/.bashrc
  echo 'export CCACHE_DIR=~/.ccache' >> ~/.bashrc
  echo 'export CCACHE_MAXSIZE=10G' >> ~/.bashrc
fi

# 初始化ccache
ccache -M 10G || true

echo "=== OpenWrt 编译环境安装完成 ==="
echo "请执行：source ~/.bashrc"
echo "然后开始编译 OpenWrt"
```

脚本2：一键清理 / 卸载环境

文件名建议：`openwrt-env-clean.sh`

```
#!/usr/bin/env bash

set -e

echo "=== 开始清理 OpenWrt 编译环境 ==="

# 删除编译目录
rm -rf ~/openwrt-build

# 清理ccache
ccache -C || true
rm -rf ~/.ccache

# 删除环境变量
sed -i '/USE_CCACHE/d' ~/.bashrc
sed -i '/CCACHE_DIR/d' ~/.bashrc
sed -i '/CCACHE_MAXSIZE/d' ~/.bashrc

# 可选：卸载依赖（⚠️谨慎）
read -p "是否卸载编译依赖？(y/N): " confirm

if [[ "$confirm" == "y" || "$confirm" == "Y" ]]; then
  sudo apt remove --purge -y \
  build-essential clang flex bison g++ gawk gcc-multilib g++-multilib \
  gettext git libncurses5-dev libssl-dev python3-distutils rsync unzip zlib1g-dev \
  file wget python3 python3-pip python3-setuptools \
  libelf-dev libglib2.0-dev libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev \
  libncursesw5-dev libreadline-dev libsqlite3-dev libtool \
  lzop patch pkgconf swig texinfo time xsltproc \
  ccache curl subversion

  sudo apt autoremove -y
  sudo apt clean
fi

echo "=== 清理完成 ==="
```

使用方法（很简单）

1.安装环境

```
chmod +x openwrt-env-install.sh
./openwrt-env-install.sh
source ~/.bashrc
```

2.清理环境（测试不满意用这个命令还原干净系统）

```
chmod +x openwrt-env-clean.sh
./openwrt-env-clean.sh
```

开始编译

1，拉源码

```
cd ~/openwrt
git tag | tail -20
git checkout v25.12.0-rc1
cd immortalwrt
```

按需要添加插件：

lucky

```
echo "src-git lucky https://github.com/gdy666/luci-app-lucky.git" >> feeds.conf.default
```

netspeedtest

```
git clone --depth 1 https://github.com/muink/luci-app-netspeedtest.git package/luci-app-netspeedtest
```

2,更新 feeds
./scripts/feeds update -a
./scripts/feeds install -a

```
make defconfig
```

坑的地方来了，报错：

```
WARNING: Makefile 'package/feeds/packages/onionshare-cli/Makefile' has a dependency on 'python3-pysocks', which does not exist WARNING: Makefile 'package/feeds/packages/onionshare-cli/Makefile' has a dependency on 'python3-unidecode', which does not exist # # configuration written to .config #
```

直接删掉这个包

```
sed -i '/onionshare-cli/d' .config
```

```
rm -rf package/feeds/packages/onionshare-cli
```

```
make defconfig
make menuconfig
```

Target System   --->选架构-x86

Subtarget   --->x86_64

Target Profile   --->Generic x86/64

Target Images  --->Root filesystem partition size (in MiB)储存容量大小（我填：2048，按实际需要填）

在 **Target Images** 菜单里，建议只保留这些：

```
[*] squashfs
[*] Build GRUB images (Linux x86 or x86_64 host only)
[*] Build GRUB EFI images          ← 这个必须选
[*] Use Console Terminal (in addition to Serial)
[*] GZip images                    ← 这个必须选（才会生成 .gz）
```

其他的可以取消勾选（用空格键切换）：

* [ ] tar.gz（不需要的话去掉）
* [ ] erofs
* [ ] ext4

* 各种虚拟机镜像（QCOW2、VDI、VMDK、VHDX、ISO）都可以去掉

Base system  --->wget-ssl（勾选）

firewall4

dnsmasq-full （全选）

opkg

Kernel modules  --->
Network Devices  --->
<*> kmod-mlx5-core*

LuCI  --->

luci-app-advanced-reboot

luci-app-appfilter

luci-app-attendedsysupgrade

luci-app-autoreboot

luci-app-commands

luci-app-ddns-go

luci-app-dockerman

luci-app-lucky

luci-app-netdata

luci-app-netspeedtest

luci-app-openclash

luci-app-nlbwmon

Include rclone-webui  #默认

Include rclone-ng (another webui)   #默认

luci-app-sqm

luci-app-tailscale-community

luci-app-ttyd

luci-app-unblockneteasemusic

luci-app-upnp

luci-app-usb-printer

luci-app-vlmcsd

luci-app-wol

Utilities --->
<*> pciutils

如果找不到主页直接用/搜索

下一步直接干

```
tmux new -s openwrt
```

```
make -j$(nproc) V=s 2>&1 | tee build.log
```

或者下面的命令只打印错误日志,方便查找:

```
make -j$(nproc) V=s 2>&1 | tee build_error.log
```

编译失败时可以搜索真正的失败信息

```
grep -n -E "ERROR:|failed to build|Error 1|Error 2|make\[.*\]: \*\*\*" build_error.log | tail -30
```

每一部分在干嘛?

<pre class="overflow-visible! px-0!" data-start="219" data-end="246"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼd">make</span><span> </span><span class="ͼf">-j</span><span>$(nproc)</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

自动用全部 CPU 线程并发编译（OK）

<pre class="overflow-visible! px-0!" data-start="272" data-end="287"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼe">V</span><span class="ͼ8">=</span><span>s</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

显示完整编译日志（非常重要，出错能定位）

<pre class="overflow-visible! px-0!" data-start="313" data-end="345"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼb">2</span><span>>&1 | </span><span class="ͼd">tee</span><span> build.log</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

关键点：

* `2>&1` → 把错误输出合并进正常输出
* `tee build.log` → 一边显示，一边写日志文件

断线恢复（tmux核心用法）

现在可以断开ssh或电脑休眠了

重新连接后：

<pre class="overflow-visible! px-0!" data-start="650" data-end="684"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span>tmux attach </span><span class="ͼf">-t</span><span> openwrt</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

直接回到编译现场，不会中断

如果编译失败（超常见）

继续编译用这个：

<pre class="overflow-visible! px-0!" data-start="738" data-end="762"><div class="relative w-full mt-4 mb-1"><div class=""><div class="relative"><div class="h-full min-h-0 min-w-0"><div class="h-full min-h-0 min-w-0"><div class="border border-token-border-light border-radius-3xl corner-superellipse/1.1 rounded-3xl"><div class="h-full w-full border-radius-3xl bg-token-bg-elevated-secondary corner-superellipse/1.1 overflow-clip rounded-3xl lxnfua_clipPathFallback"><div class="pointer-events-none absolute inset-x-4 top-12 bottom-4"><div class="pointer-events-none sticky z-40 shrink-0 z-1!"><div class="sticky bg-token-border-light"></div></div></div><div class=""><div class="relative z-0 flex max-w-full"><div id="code-block-viewer" dir="ltr" class="q9tKkq_viewer cm-editor z-10 light:cm-light dark:cm-light flex h-full w-full flex-col items-stretch ͼ5 ͼj"><div class="cm-scroller"><div class="cm-content q9tKkq_readonly"><span class="ͼd">make</span><span> </span><span class="ͼf">-j1</span><span> </span><span class="ͼe">V</span><span class="ͼ8">=</span><span>s</span></div></div></div></div></div></div></div></div></div><div class=""><div class=""></div></div></div></div></div></pre>

单线程可以：

* 清晰报错
* 不会乱日志

如果你看到：

```
Downloading...
Cloning...
fatal: unable to access
```

是网络问题,等下重试，要不换个好点的网络，要不云编译，没其他方法。

阶段1：下载阶段（需要网络）

make download

阶段2：编译阶段

make -j4

阶段3：特殊包（少数情况）

比如：

* openclash（下载核心）
* 某些 Go / Rust 包
* npm / pip 依赖
* 才会临时访问外网

25.12.0版本以后该用apk安装插件方式，weiui上传第三方插件会无法安装，提示失败后可以到ssh安装：

```
cd /tmp
apk add --allow-untrusted /tmp/upload.apk
```

或者

```
apk add --allow-untrusted /tmp/upload.apk
```
