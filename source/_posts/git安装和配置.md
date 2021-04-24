---
title: git安装和配置
date: 2021-03-29 21:39:25
tags: git
---
# git安装和配置

## 1. 版本控制工具

**SVN**：集中式版本控制工具

集中式版本控制工具，几乎所有的动作**都需要服务器参与**，并且**数据安全性与服务器关系很大**。

**git**：分布式版本控制工具

**Git 是分布式版本控制工具**，除了与服务器之前进行按需同步之外，**所有的提交操作都不需要服务器**。

**SVN**不适用的领域：

- 跨地域协同开发
- 追修高质量代码以及代码门禁

**git**不适合的领域：

- 不适合Word等二进制文档的版本控制
- git没有锁定/解锁模式，不能派他式修改
- 权限不能细分（例如将不同的文件分配不同的访问权限）

## 2. git安装和配置

### 2.1 Linux下安装 git

**包管理器方式安装**

Ubuntu 10.10(maverick)或更新版本，Debian(squeeze)或更新版本:

```bash
$ sudo aptitude install git
$ sudo aptitude install git-doc git-svn git-email gitk
```

Linux 系统:  RHEL、Fedora、CentOS 等版本:

```bash
$ yum install git
$ yum install git-svn git-email gitk
```

其中`git`软件包包含了大部分Git命令，是必装的软件包。

软件包`git-svn`、`git-email`、`gitk`本来也是Git软件包的一部分，但是因为有着不一样的软件包依赖（如更多`perl`模组，`tk`等），所以单独作为软件包发布。

**源码安装**

访问http://git-scm.com/，下载Git源码包，例如：git-2.19.0.tar.gz。

第一步：展开源码包，并进入到相应的目录中。

```bash
$ tar -jxvf git-2.19.0.tar.bz2
$ cd git-2.19.0
```

第二步：安装。

```bash
$ make prefix=/usr/local all
$ sudo make prefix=/usr/local install
```

第三步：安装git文档（可选）

```bash
$ make prefix=/usr/local doc info
$ sudo make prefix=/usr/local install-doc install-html install-info
```

### 2.2 **Windows** **下安装** git

**下载安装包**

[https://git-scm.com/download/win ](https://git-scm.com/download/win)下载 Windows 安装包

**安装安装包**

![image-20210329144403430](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210329144403430.png)

在这一步去掉git LFS选项。

![image-20210329144502858](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210329144502858.png)

在这一步可以选择编辑器，默认就可以。

![image-20210329144551825](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210329144551825.png)

这一步选择git bash作为命令行。

**TortoiseGit**

**TortoiseGit**是一个git的可视化图形界面。个人认为用处不大。



### 2.3 git安装完成后的配置

git的配置分为三类，系统配置（适用所有用户），用户配置（适用于该用户），仓库配置（只对当前项目有效）。

**系统配置**（对所有用户都适用）

存放在git的安装目录下：`%Git%/etc/gitconfig`；若使用 `git config` 时用` --system` 选项，读写的就是这个文件：

`git config --system core.autocrlf`

**用户配置**（只适用于该用户）

存放在用户目录下。例如linux存放在：`~/.gitconfig`；若使用 `git config` 时用` --global` 选项，读写的就是这个文件：

`git config --global user.name`

**仓库配置**（只对当前项目有效）

当前仓库的配置文件（也就是工作目录中的 `.git/config` 文件）；若使用`git config` 时用 `--local` 选项，读写的就是这个文件：

`git config --local remote.origin.url`



**配置第一步：配置用户信息**

```bash
git config --global user.name “XXXXX”
git config --global user.email XXXXXXXXXX@huawei.com
```

第一行设置自己git提交时的姓名，第二行设置提交时的邮箱。

**配置第二步：配置文本换行符**

原因：Windows时用回车和换行两个字符来换行，Linux只使用换行。如果涉及到多平台开发，就可能造成问题。

```bash
$ git config --global core.autocrlf true
```

在Windows下使用这个命令就可以让git在提交时自动帮忙转化。

**配置第三步：文本编码配置**

```bash
\# 中文编码支持
git config --global gui.encoding utf-8
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8

\# 显示路径中的中文：
git config --global core.quotepath false
```

- **i18n.commitEncoding** **选项：**用来让git commit log存储时，采用的编码，默认UTF-8.

- **i18n.logOutputEncoding** **选项：**查看git log时，显示采用的编码，建议设置为UTF-8.

**配置第四步：生成ssh认证文件**

> 认证：在与远程的git服务器，例如GitHub，GitLab进行通讯时，建立起来的认证，证明自己可以在当前这个机器上对远程仓库的内容进行修改。

两种认证方法：

1. http/https协议认证（不推荐使用，因此不写了）
2. ssh协议认证

ssh协议认证

```bash
$ ssh-keygen -t rsa –C zhangsan1123@huawei.com
```

在git bash中输入这段命令，敲击几次回车，会在计算机中生成ssh认证文件。

ssh认证文件存放地址：

Windows ：C:\Users\Administrator\\.ssh    *Administrator为你当前登陆的用户名*

Linux： ~/.ssh           *~为当前用户的家目录*

打开文件夹，找到`id_rsa.pub`文件，打开复制内部的内容。

添加公钥到代码平台：

1. 登录代码平台

2. 进入“Profile Settings”

3. 点击左侧栏的“SSH Keys”

4. 点击“Add SSH Key”,将刚生成的公钥文件的内容，复制到“Public Key”栏，保存即可。