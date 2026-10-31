---
title: Emby网页端在 Linux上调用 MPV和 VLC：排查与修复记录
date: 2026-10-31
featured_image: Pasted%20image%2020261031193300.png
featured_image_preview: Pasted%20image%2020261031193300.png
draft: false
tags:
  - Emby
  - Linux
  - MPV
  - VLC
---
## 环境与症状

Emby 服务运行在 amilys 的 Emby Docker 镜像中，外部播放器按钮由 `embyExternalUrl` / `embyLaunchPotplayer.js` 网页脚本提供。浏览器运行在 Fedora 桌面，Docker 容器运行在另一台 Ubuntu 主机上。

最初点击 Emby 的外部播放时，浏览器询问是否打开 `xdg-open`；继续后 GNOME 显示“无可用应用”。MPV 和 VLC 都已安装。排查中先恢复 MPV，再修复 VLC 的 Linux 协议和桌面启动项。

> 下文里的容器名、服务器地址、端口、媒体 ID、API 令牌、设备 ID 和服务器 ID 都是示例或占位符。完整播放链接包含 API 令牌，不能公开粘贴到聊天、日志或文章中。

## 排查流程

### 1. 确认播放器是否安装

在运行 Emby 网页浏览器的 Linux 主机上执行：

```bash
command -v mpv
command -v vlc
```

本次两条命令都找到了可执行文件，说明播放器本体存在。若返回空，先安装相应播放器，再继续排查。

### 2. 检查常规视频默认应用和协议默认应用

执行：

```bash
xdg-mime query default video/mp4
xdg-mime query default video/x-matroska
xdg-mime query default x-scheme-handler/mpv-handler
xdg-mime query default x-scheme-handler/vlc
```

本次视频 MIME 类型默认应用是 GNOME Showtime；MPV 协议则已指向 `mpv-handler.desktop`。常规视频默认应用只有在程序通过视频文件 MIME 类型打开时才相关；Emby 外部播放实际使用自定义 URL 协议，所以还需要单独核对协议处理器。

需要检查用户级 MIME 配置和启动项时：

```bash
cat ~/.config/mimeapps.list
cat ~/.local/share/applications/mpv-handler.desktop
```

### 3. 确认 Emby 容器实际使用的脚本版本和 MPV 协议

先在运行 Docker 的 Ubuntu 主机上查看容器名：

```bash
docker ps --format 'table {{.Names}}\t{{.Image}}'
```

查找容器中外部播放脚本的版本和协议字符串：

```bash
docker exec <容器名> sh -lc \
'grep -RnoE "@version[[:space:]]+[0-9.]+|mpv-handler://play/|mpv://play/" /system/dashboard-ui /config/config 2>/dev/null | head -40'
```

本次结果显示脚本版本是 `1.1.0`，并生成旧的 `mpv://play/`。本机 `mpv-handler` 注册的是 `mpv-handler://`，两端协议不一致。更新到 1.2.1 后，脚本改为生成 `mpv-handler://play/`，MPV 恢复正常。

### 4. 更新 Emby 脚本，并保留备份

先备份旧脚本：

```bash
docker cp <容器名>:/system/dashboard-ui/embyLaunchPotplayer.js \
  /tmp/embyLaunchPotplayer.js.bak
```

从插件 GitHub 仓库下载最新版脚本：

```bash
curl -fL \
  https://raw.githubusercontent.com/bpking1/embyExternalUrl/main/embyWebAddExternalUrl/embyLaunchPotplayer.js \
  -o /tmp/embyLaunchPotplayer.js
```

下载后先检查版本和 MPV 协议，再复制进容器：

```bash
grep -nE '@version|mpv-handler://play/' /tmp/embyLaunchPotplayer.js | head

docker cp /tmp/embyLaunchPotplayer.js \
  <容器名>:/system/dashboard-ui/embyLaunchPotplayer.js

docker exec <容器名> grep -nE '@version|mpv-handler://play/' \
  /system/dashboard-ui/embyLaunchPotplayer.js | head
```

本次容器内没有 `python3`。尝试在容器内用 Python 修改时得到 `python3: executable file not found in $PATH`，文件没有被修改。备份已经保留，因此改为把文件复制到宿主机，在宿主机用 `sed` 修改，再复制回容器。

### 5. 查明并修复 VLC 的 Linux 协议分支

查看 `embyVlc()` 函数附近的代码：

```bash
docker exec <容器名> sed -n '390,420p' \
  /system/dashboard-ui/embyLaunchPotplayer.js
```

发现 VLC 链接默认使用 Android 的 `intent:`；只有 Windows 和 macOS 分支会改用 `vlc://`。在桌面 Linux 上也需要生成 `vlc://`。

先确认旧条件只匹配一次：

```bash
grep -Fc 'if (OS.isWindows() || OS.isMacOS()) {' \
  /tmp/embyLaunchPotplayer.js.bak
```

本次结果为 `1`。然后在宿主机上编辑备份副本：

```bash
cp /tmp/embyLaunchPotplayer.js.bak /tmp/embyLaunchPotplayer.js

sed -i \
  's/if (OS.isWindows() || OS.isMacOS()) {/if (!OS.isAndroid() \&\& !OS.isIOS()) {/' \
  /tmp/embyLaunchPotplayer.js
```

核对修改结果：

```bash
grep -nE 'isAndroid\(\).*isIOS\(\)|OS.isWindows\(\).*OS.isMacOS\(\)' \
  /tmp/embyLaunchPotplayer.js
```

本次 VLC 分支显示 `if (!OS.isAndroid() && !OS.isIOS())`。将修改后的文件复制回容器：

```bash
docker cp /tmp/embyLaunchPotplayer.js \
  <容器名>:/system/dashboard-ui/embyLaunchPotplayer.js
```

### 6. 在 Fedora 注册 `vlc://` 协议处理器

以下命令要在**运行浏览器的 Fedora 主机**执行，而不是 Docker 容器内：

```bash
sudo curl -fL \
  https://raw.githubusercontent.com/northsea4/vlc-protocol/main/linux/vlc-protocol \
  -o /usr/local/bin/vlc-protocol
sudo chmod +x /usr/local/bin/vlc-protocol

curl -fL \
  https://raw.githubusercontent.com/northsea4/vlc-protocol/main/linux/vlc-protocol.desktop \
  -o /tmp/vlc-protocol.desktop
xdg-desktop-menu install /tmp/vlc-protocol.desktop
xdg-mime default vlc-protocol.desktop x-scheme-handler/vlc
xdg-mime query default x-scheme-handler/vlc
```

最后一条返回 `vlc-protocol.desktop`，证明 MIME 默认应用已登记。该协议处理器负责去掉 `vlc://` 前缀并把媒体地址交给 VLC；它也包含 Chromium 把嵌套 `http://` 改写成 `http//` 时的修复逻辑。

### 7. 用 F12 确认网页实际生成的 VLC 链接

在 Emby 页面按 F12 打开开发者工具，选择 Console，点击 VLC 按钮。插件会打印生成的链接。本次链接以 `vlc://http//...` 开头，浏览器日志显示：

```text
Launched external handler for 'vlc://http//<服务器地址>/<播放路径>?...'
```

Chromium 会移除嵌套 `http://` 的冒号；`vlc-protocol` 应将 `http//` 修正回 `http://`。分享这类日志时，必须遮住 API 令牌、服务器地址、设备 ID、媒体 ID 等内容，只保留协议名和参数结构。

### 8. 分层验证协议处理器和媒体地址

先做本机协议启动冒烟检查：

```bash
xdg-open 'vlc://file:///etc/hostname'
```

本次 VLC 能启动。此检查只验证桌面协议分发及 VLC 启动，不验证 Emby 媒体地址。

再用完整 Emby 链接分别测试处理器和桌面分发。真实命令只在本机终端输入，不要把带真实令牌的命令输出发出来：

```bash
/usr/local/bin/vlc-protocol 'vlc://http//<服务器地址>/<播放路径>?api_key=<令牌>&Static=true&MediaSourceId=<媒体源ID>&DeviceId=<设备ID>'

xdg-open 'vlc://http//<服务器地址>/<播放路径>?api_key=<令牌>&Static=true&MediaSourceId=<媒体源ID>&DeviceId=<设备ID>'
```

两种方式都能打开视频，证明 VLC、协议脚本、桌面 MIME 分发和 Emby 媒体链接本身都能工作。问题只在 Emby 网页经 Chrome 启动处理器时仍出现“无可用应用”提示。

### 9. 给桌面启动项设置绝对路径

将处理器直接执行和 `xdg-open` 都验证成功后，仍需修复 Chrome 发起的桌面启动。用户级启动项明确写出处理器的绝对路径：

```bash
mkdir -p ~/.local/share/applications

cat > ~/.local/share/applications/vlc-protocol.desktop <<'DESKTOP'
[Desktop Entry]
Version=1.0
Name=VLC protocol
Comment=Open vlc:// links with VLC
Exec=/usr/local/bin/vlc-protocol %u
TryExec=/usr/local/bin/vlc-protocol
Icon=vlc
Terminal=false
Type=Application
MimeType=x-scheme-handler/vlc;
NoDisplay=true
DESKTOP

update-desktop-database ~/.local/share/applications
xdg-mime default vlc-protocol.desktop x-scheme-handler/vlc
xdg-mime query default x-scheme-handler/vlc
```

然后完全退出 Chrome 并重新打开，再从 Emby 点击 VLC。此后 MPV 和 VLC 均可正常播放。经验上，`xdg-mime query` 只确认默认关联；直接运行处理器、`xdg-open` 完整媒体链接、最后在浏览器中点击，是三个不同层次的验证，最好逐层进行。

## 回滚

恢复 Emby 容器中的原始脚本：

```bash
docker cp /tmp/embyLaunchPotplayer.js.bak \
  <容器名>:/system/dashboard-ui/embyLaunchPotplayer.js
```

移除 Fedora 用户级 VLC 启动项后，可重新使用系统级启动项：

```bash
rm ~/.local/share/applications/vlc-protocol.desktop
update-desktop-database ~/.local/share/applications
xdg-mime default vlc-protocol.desktop x-scheme-handler/vlc
```

## 后续维护

- Emby 容器更新或重建可能覆盖手工复制到 `/system/dashboard-ui/` 的脚本。更新后重新检查版本、`mpv-handler://play/` 协议及 VLC 分支。
- 更新前保留脚本备份，并记录实际运行命令的主机：Docker 文件操作在 Ubuntu 主机执行；VLC 处理器和 MIME 注册在运行浏览器的 Fedora 主机执行。
- 调试时分享浏览器日志或播放链接，务必先删除 API 令牌和其他内部地址/标识。
