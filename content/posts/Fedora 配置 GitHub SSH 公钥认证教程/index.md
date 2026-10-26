---
title: Fedora 配置 GitHub SSH 公钥认证教程
draft: false
featured: false
date: 2026-10-26T18:28:18+08:00
featured_image: Pasted%20image%2020261026183124.png
featured_image_preview: Pasted%20image%2020261026183124.png
summary: ""
categories:
  - tools
tags:
  - ssh
  - GitHub
---
部署测试hugo，hexo等博客到github必需设置好ssh公钥，省去每次输入用户密码的麻烦，节省不少时间，一次设好，以后都不需要麻烦了。
<!--more-->
## 一、为什么要配置 SSH 公钥？

使用 GitHub 管理 Hugo 博客时，可以通过 HTTPS 或 SSH 与远程仓库通信。

HTTPS 推送可能遇到权限验证或令牌配置问题。配置 SSH 公钥后，可以使用 SSH 密钥对验证身份，避免每次推送都处理 HTTPS 凭据。

SSH 认证使用一对密钥：

- **私钥**：保存在本地电脑，不能泄露给任何人。
    
- **公钥**：添加到 GitHub 账户，用于验证本地电脑的身份。
    

本文以 Fedora Linux 为例，演示如何配置 GitHub SSH 认证。

## 二、检查 SSH 工具

打开 Fedora 终端，执行：

```
ssh -V
```

如果能够显示 OpenSSH 版本，说明 SSH 客户端已经安装。

如果提示找不到命令，可以安装：

```
sudo dnf install openssh-clients git
```

## 三、生成 SSH 密钥

### 1. 创建专用密钥

建议为 GitHub 单独创建一组 SSH 密钥，使用 Ed25519 算法：

```
ssh-keygen -t ed25519 -C "你的GitHub邮箱" -f ~/.ssh/id_ed25519_github_fedora
```

请将 `你的GitHub邮箱` 替换成自己使用的 GitHub 邮箱。

命令执行后，会询问是否设置密钥密码（passphrase）。

- 设置密码：安全性更高，使用密钥时可能需要输入密码或借助 SSH Agent。
    
- 直接回车：不设置密码，使用更方便，但需要格外保护好本地账户和私钥。
    

生成完成后，会得到两个文件：

|文件|作用|
|---|---|
|`~/.ssh/id_ed25519_github_fedora`|私钥，只保存在本地|
|`~/.ssh/id_ed25519_github_fedora.pub`|公钥，可以添加到 GitHub|

**注意：不要上传、发送或公开没有** `**.pub**` **后缀的私钥文件。**

### 2. 检查密钥是否生成

执行：

```
ls -l ~/.ssh/id_ed25519_github_fedora*
```

正常情况下会看到公钥和私钥两个文件。

## 四、将公钥添加到 GitHub

### 1. 复制公钥内容

在 Fedora 终端执行：

```
cat ~/.ssh/id_ed25519_github_fedora.pub
```

终端会显示一整行以 `ssh-ed25519` 开头的内容。

复制**完整的一行公钥**，不要复制私钥。

### 2. 打开 GitHub SSH 设置

访问：

[https://github.com/settings/keys](https://github.com/settings/keys)

按照以下步骤操作：

1. 登录 GitHub。
    
2. 点击 **New SSH key**。
    
3. 在 Title 中填写容易辨认的名称，例如 `Fedora HP Elite x2 G4`。
    
4. Key type 选择 `Authentication Key`。
    
5. 将刚才复制的公钥粘贴到 Key 输入框。
    
6. 点击 **Add SSH key** 完成添加。
    

如果 GitHub 要求重新验证账户身份，按照页面提示操作即可。

## 五、测试 SSH 连接

### 1. 执行连接测试

在 Fedora 终端运行：

```
ssh -T -i ~/.ssh/id_ed25519_github_fedora git@github.com
```

首次连接时，可能出现类似下面的提示：

```
The authenticity of host 'github.com' can't be established.
```

确认主机指纹与 GitHub 官方公布的 SSH 主机指纹一致后，再输入：

```
yes
```

GitHub 官方 SSH 指纹说明：

[https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)

如果连接成功，会看到类似下面的提示：

```
Hi YOUR_USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

这表示 SSH 身份认证成功。GitHub 不提供普通 SSH 终端登录，因此提示 `does not provide shell access` 是正常现象。

## 六、将 Git 仓库切换为 SSH 地址

如果之前使用 HTTPS 克隆或配置仓库，需要将远程地址改为 SSH 格式。

### 1. 进入 Hugo 项目目录

例如：

```
cd ~/hugo-migration/newsqualychen.github.io
```

请根据自己的实际项目路径调整。

### 2. 检查当前远程地址

```
git remote -v
```

如果显示类似：

```
origin  https://github.com/用户名/仓库名.git (fetch)
origin  https://github.com/用户名/仓库名.git (push)
```

说明目前使用的是 HTTPS。

### 3. 修改为 SSH 地址

以个人博客仓库 `newsqualychen.github.io` 为例：

```
git remote set-url origin git@github.com:newsqualychen/newsqualychen.github.io.git
```

然后再次检查：

```
git remote -v
```

正常情况下会显示：

```
origin  git@github.com:newsqualychen/newsqualychen.github.io.git (fetch)
origin  git@github.com:newsqualychen/newsqualychen.github.io.git (push)
```

请将用户名和仓库名替换为自己的实际信息。

## 七、让 Git 自动使用指定的 SSH 密钥

如果你只配置了一组 GitHub SSH 密钥，通常可以直接使用。

如果本地存在多组 SSH 密钥，建议为 GitHub 指定密钥，避免 SSH 自动选择了其他密钥。

### 1. 编辑 SSH 配置文件

```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/config
```

在文件中添加：

```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_fedora
    IdentitiesOnly yes
```

保存并退出 Nano：

1. 按 `Ctrl + O` 保存。
    
2. 按回车确认文件名。
    
3. 按 `Ctrl + X` 退出。
    

如果配置文件已经存在，请先检查原有内容，避免覆盖其他主机的配置。

### 2. 设置配置文件权限

```
chmod 600 ~/.ssh/config
chmod 600 ~/.ssh/id_ed25519_github_fedora
chmod 644 ~/.ssh/id_ed25519_github_fedora.pub
```

### 3. 再次测试

```
ssh -T git@github.com
```

如果显示 GitHub 的认证成功提示，说明 SSH 配置已经生效。

## 八、测试 Git 推送

进入仓库目录：

```
cd ~/hugo-migration/newsqualychen.github.io
```

检查当前分支：

```
git branch --show-current
```

检查工作区状态：

```
git status
```

如果已经有提交且工作区干净，可以尝试：

```
git push
```

如果本地存在尚未提交的修改，需要先根据实际情况执行 `git add` 和 `git commit`，再推送。

推送成功后，可以到 GitHub 仓库的 **Actions** 页面查看自动部署任务是否正常运行。

对于 Hugo 博客，还需要确认 GitHub Pages 的发布来源设置为 `GitHub Actions`，并且仓库中存在正确的 Hugo 部署工作流。SSH 推送成功不代表网站构建和部署一定成功。

## 九、常见问题排查

### 问题 1：`Permission denied (publickey)`

可能原因：

- 公钥没有添加到正确的 GitHub 账户。
    
- SSH 使用了错误的私钥。
    
- SSH 配置文件中的密钥路径不正确。
    
- 远程仓库地址或账户权限有误。
    

先执行：

```
ssh -vT -i ~/.ssh/id_ed25519_github_fedora git@github.com
```

观察调试输出中是否尝试使用了正确的密钥。

同时确认公钥内容：

```
cat ~/.ssh/id_ed25519_github_fedora.pub
```

与 GitHub SSH Keys 页面中的密钥一致。

### 问题 2：`Repository not found` 或没有推送权限

检查远程地址：

```
git remote -v
```

确认仓库名称、GitHub 用户名是否正确，并确认当前账户具有该仓库的写入权限。

### 问题 3：修改 SSH 地址后仍然提示 HTTPS 认证失败

检查当前仓库是否确实使用 SSH：

```
git remote get-url origin
```

如果输出仍以 `https://` 开头，重新设置远程地址。

### 问题 4：忘记之前生成的密钥保存在哪里

查看 SSH 目录：

```
ls -lah ~/.ssh
```

可以检查文件名，但不要随意删除未知密钥，因为其他仓库或服务器可能仍在使用它们。

## 十、安全建议

1. 私钥文件绝不能上传到 GitHub 仓库。
    
2. 不要把私钥粘贴到聊天窗口、博客文章或公开截图中。
    
3. `.pub` 公钥可以公开，但建议只添加到可信的 GitHub 账户。
    
4. 如果怀疑私钥泄露，应尽快在 GitHub 的 SSH Keys 页面撤销对应公钥，并生成新的密钥对。
    
5. 不要为了绕过连接问题而关闭 SSH 主机指纹校验。
    
6. 在多台电脑上管理 Hugo 博客时，可以为每台电脑分别生成 SSH 密钥，并给公钥设置清晰的名称，方便日后管理。
    

## 十一、总结

完成以上步骤后，Fedora 就可以通过 SSH 与 GitHub 通信：

1. 生成 SSH 密钥对。
    
2. 将 `.pub` 公钥添加到 GitHub。
    
3. 使用 `ssh -T git@github.com` 测试身份认证。
    
4. 将 Git 远程仓库地址改为 SSH 格式。
    
5. 使用 `git push` 推送 Hugo 博客源文件。
    
6. 在 GitHub Actions 中检查 Hugo 构建和 GitHub Pages 部署结果。
    

配置完成后，日常更新博客通常只需要在仓库中提交修改，再执行 `git push` 即可。