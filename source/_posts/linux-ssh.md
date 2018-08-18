---
title: Linux学习笔记-SSH
date: 2018-07-09
tags: [linux]
categories: 运维知识
---

# 准备工作

## 替换默认源

- http://mirrors.163.com/.help/centos.html
- 腾讯云主机默认为腾讯源无需操作

## SSH 工具

- SSH config 用法详解
- 免密码登陆方案之 SSH Key
- SSH 端口安全
- 个性化脚本一键登录服务器

### SSH 是什么

![ssh.png](https://upload-images.jianshu.io/upload_images/4869616-2ffc372b5d41e6f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- Secure Shell 安全外壳协议
- 建立在应用层基础上的安全协议
- 可靠，专为远程登录会话和其他网络服务提供安全性的协议
- 有效防止远程管理过程中的信息泄露问题
- 适用于多种平台，几乎支持所有的 UNIX 平台

### 服务器安装 SSH 服务（服务器版本已安装）

- 安装 SSH

```
yum install openssh-server
```

- 启动 SSH

```
service sshd start
```

- 设置开机运行

```
chkconfig sshd on
```

### 客户端安装 SSH 工具

- SSH 是典型的客户端与服务端的交互模式
- win: Xshell/Putty/secureCRT
- mac: 系统自带,在终端执行如下命令即可

```
ssh root@your-server-ip
```

使用 oh-my-zsh 的同学可以更新~/.zshrc 文件加入如下别名

```
alias ssh_centos="ssh root@your-server-ip
```

后续只需使用`ssh_centos`命令,输入密码即可登陆云主机

### 验证

从客户端来看，SSH 提供两种级别的安全验证。

- 第一种级别（基于口令的安全验证）

只要你知道自己帐号和口令，就可以登录到远程主机。所有传输的数据都会被加密，但是不能保证你正在连接的服务器就是你想连接的服务器。可能会有别的服务器在冒充真正的服务器，也就是受到“中间人”这种方式的攻击。

- 第二种级别（基于密匙的安全验证）

需要依靠密匙，也就是你必须为自己创建一对密匙，并把公用密匙放在需要访问的服务器上。如果你要连接到 SSH 服务器上，客户端软件就会向服务器发出请求，请求用你的密匙进行安全验证。服务器收到请求之后，先在该服务器上你的主目录下寻找你的公用密匙，然后把它和你发送过来的公用密匙进行比较。如果两个密匙一致，服务器就用公用密匙加密“质询”（challenge）并把它发送给客户端软件。客户端软件收到“质询”之后就可以用你的私人密匙解密再把它发送给服务器。
用这种方式，你必须知道自己密匙的口令。但是，与第一种级别相比，第二种级别不需要在网络上传送口令。

![基于密钥的安全验证.png](https://upload-images.jianshu.io/upload_images/4869616-a05ee38b13ec99b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二种级别不仅加密所有传送的数据，而且“中间人”这种攻击方式也是不可能的（因为他没有你的私人密匙）。但是整个登录的过程可能需要 10 秒

**其实每一个新手配置 github 的过程都走过这些路**

### SSH config 讲解

- config 为了方便我们**批量**管理 ssh（如在同一主机上配置 github 与 gitlab）
- config 存放在~/.ssh/config
- config 配置语法

| 英文         | 意义           |
| ------------ | -------------- |
| Host         | 别名           |
| HostName     | 主机名         |
| Port         | 端口(默认 22)  |
| User         | 用户名         |
| IdentityFile | 密钥文件的路径 |

举例：

1.  公司电脑配置 github 与 gitlab 配置文件

```
# gitlab
Host gitlab
	HostName gitlab.com
	IdentityFile ~/.ssh/id_rsa

# github
Host github
	HostName github.com
	IdentityFile ~/.ssh/github_id_rsa
```

举例：

2.  家里电脑配置 github 与腾讯云主机

```
# github
Host github
	HostName github.com
	IdentityFile ~/.ssh/id_rsa
# tencentServer
Host centos
	HostName your-server-ip
	User root
	Port 22
```

这样即使不通过别名，也可以直接使用`ssh centos`直接连接主机。
注意，我们虽然没有如同腾讯云主机一般对 github 以及 gitlab 的主机进行连接，其实 push 的过程就是连接主机的过程，同时 github 以及 gitlab 都配置了 ssh key，即`IdentityFile`，连接（push）的时候无需输入密码，但是我们的腾讯云主机此时还是仅仅配置了一个别名，还未生成 ssh key 进行验证,依旧需要输入密码。

### 免密码登陆方案之 SSH Key

- ssh key 使用非对称加密方式生成公钥和私钥
- 私钥存放在本地~/.ssh 目录
- 公钥可以对外公开，放在服务器的~/.ssh/authorized_keys

linux 平台生成 ssh key 的命令

```
ssh-keygen -t rsa
```

注意：

生成的时候不要和已有的 id_rsa 文件重复，否则会覆盖，生成过程中有一个阶段会询问文件命名。

由于我之前配置过 github 的 ssh，本地`~/.ssh`文件夹内已经存在`id_rsa`以及`id_rsa.pub`，即默认命名，所以在生成云主机 ssh key 的时候要注意。

1.  公钥：将带有.pub 后缀的公钥文件存放在云主机`~/.ssh/authorized_keys`文件内（github 则是直接复制在网站对应入口）
2.  私钥：更新我们的 config 文件(如果只有一个服务，是不需要 config 文件的)，并使用`ssh-add`命令载入

```
# github
Host github
	HostName github.com
	IdentityFile ~/.ssh/id_rsa
# tencentServer
Host centos
	HostName your-server-ip
	IdentityFile ~/.ssh/centos_id_rsa
	User root
	Port 22
```

```
ssh-add ~/.ssh centos_id_rsa
```

3.  现在直接通过`ssh centos`即可无需密码登陆云主机

### ssh 安全端口

- 端口安全指的是尽量避免服务器的远程连接端口被不法分子知道，为此而改变默认服务端口号的操作
- 如何修改 ssh 服务端口，修改如下文件

```
/etc/ssh/sshd_config
```

![/etc/ssh/sshd_config.png](https://upload-images.jianshu.io/upload_images/4869616-07d06f46082ddb14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 可以同时监听多个端口
- 修改完文件后记得重启服务

```
service sshd restart
```
