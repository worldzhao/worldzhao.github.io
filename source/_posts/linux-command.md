---
title: Linux学习笔记-常用命令
date: 2018-07-08
tags: [linux]
categories: 运维知识
---

# Linux 常用命令

- 软件操作命令
- 服务器硬件资源信息
- 文件和文件夹操作命令
- 系统用户操作命令
- 防火墙相关设置
- 提权操作 sudo 和文件传输操作

## 软件操作命令

- 软件包管理器：yum
- 安装软件命令：yum install xxx
- 卸载软件命令：yum remove xxx
- 搜索软件命令：yum search xxx
- 清理缓存命令：yum clean packages
- 列出已安装： yum list
- 软件包信息： yum info xxx

## 服务器硬件资源信息

- 内存：free -m
- 磁盘：df -h
- 负载：w/top
- cpu 信息：cat /proc/cpuinfo

## 文件操作命令

- Linux 文件的目录结构
- 文件基本操作
- 文本编辑器 vim
- 文件权限 4-2-1
- 文件的搜索/查找/读取
- 文件的压缩或解压

### Linux 文件的目录结构

![linux目录结构.png](https://upload-images.jianshu.io/upload_images/4869616-bbeee309e29ef479.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 根目录 /
- 家目录 /home
- 临时目录 /tmp
- 配置目录 /etc
- 用户程序目录 /usr

### 文件基本操作

| 命令  | 解释                    |
| ----- | ----------------------- |
| ls    | 查看目录下的文件[-a -l] |
| touch | 新建文件                |
| mkdir | 新建文件夹[-p]          |
| cd    | 进入目录                |
| rm    | 删除文件和目录[-r -f]   |
| cp    | 复制                    |
| mv    | 移动(剪切)              |
| pwd   | 显示当前路径            |

### 文件编辑器 vim

| vim | 操作         |
| --- | ------------ |
| i   | 插入模式     |
| esc | 退出当前模式 |
| yy  | 复制当前行   |
| p   | 粘贴         |
| dd  | 删除         |
| u   | 恢复         |
| gg  | 到行首       |
| G   | 到行尾       |
| :wq | 写入并退出   |

### 文件权限 4-2-1

| 数字 | 权限   |
| ---- | ------ |
| 4    | r 读   |
| 2    | w 写   |
| 1    | x 执行 |

```math
2^2 + 2^1 + 2^0 = 4 + 2 + 1 = 7
```

### 文件的搜索/查找/读取

| 命令      | 解释                                   |
| --------- | -------------------------------------- |
| tail      | 从文件尾部开始读取                     |
| head      | 从文件头部部开始读取                   |
| cat       | 读取整个文件                           |
| more,less | 分页读取                               |
| grep      | 查找文件内容 grep [-n] 'hello' test.js |
| find      | 查找文件                               |

### 文件解压缩

- tar 命令

```
tar -cf archive.tar foo bar
      # 压缩         Create archive.tar from files foo and bar.

tar -tvf archive.tar
      # 查看压缩文件 List all files in archive.tar verbosely.

tar -xf archive.tar
      # 解压缩       Extract all files from archive.tar.
```

### 系统用户操作命令

| 命令             | 解释     |
| ---------------- | -------- |
| useradd username | 添加用户 |
| adduser username | 添加用户 |
| userdel username | 删除用户 |
| passwd username  | 设置密码 |

### 防火墙设置

- 作用：保护服务器安全
- 设置防火墙规则：开放 80/22 端口
- 关闭防火墙
- 安装：yum install firewalld
- 启动：service firewalld start
- 检查状态：service firewalld status
- 关闭或禁用防火墙：service firewalld stop/disbale

查看是否已经安装了防火墙

```
yum list | grep firewall
```

查看是否启动了防火墙

```
ps -ef | grep firewall
```

查看防火墙运行状态

```
firewall-cmd --state
```

查询[ssh]服务

```
firewall-cmd --query-service=ssh
```

添加[ssh]服务

```
firewall-cmd --add-service=ssh
```

移除[ssh]服务，无法访问云主机了

```
firewall-cmd --remove-service=ssh
```

查询[22]端口

```
firewall-cmd --query-port=22/tcp
```

添加[22]端口

```
firewall-cmd --add-port=22/tcp
```

移除[22]端口

```
firewall-cmd --remove-port=22/tcp
```

### 提权和文件上传下载操作

- 提权：sudo
  - visudo（给其他用户配置权限方可提权）
- server 文件下载
  - wget curl
- client 文件上传
  - scp

```
scp testUpload.txt username@your-server-ip:/tmp/
```

- client 文件下载

```
scp username@your-server-ip:/tmp/testUpload.txt ./
```

## 查看 ip

- ifconfig
- ip addr
- vi /etc/sysconfig/network-scripts/ifcfg-xx
- yum install net-tools
