# Samba 文件共享

如果有人问：“哪种文件共享支持所有主流类型的操作系统？”，我会毫不犹豫的回答 —— Samba！

Samba 是微软 CIFS	协议的开源实现，几乎所有智能路由器上的 USB 共享功能都是通过 Samba 实现的。

## 安装 Samba

在《[通过 SSH 远程管理服务器](initialization/use-ssh.md)》中的 `用主机名访问 NAS 主机` 部分已经安装了 Samba 软件包，如果没有请安装：

```
getnas@getnas:~$ sudo apt install samba
```

## 创建共享

### 第一步 创建共享目录

我们前面配置的数据盘挂载在 `/mnt/storage` 目录，虽然可以直接共享这个目录，但还是建议创建专门的文件夹进行共享，我们在该目录中创建一个 `share` 文件夹：

```
getnas@getnas:~$ mkdir /mnt/storage/share
```

### 第二步 创建 samba 用户

直接使用 Debian 系统中的用户是无法访问 Samba 共享的，我们必须在 samba 中创建与系统用户同名的用户，这里以创建 `getnas` 用户为例：

```
getnas@getnas:~$ 
```

> 注意：如果系统中不存在 `getnas` 用户，在创建 samba 用户时会报错。