# Samba 文件共享

如果有人问：“哪种文件共享支持所有主流类型的操作系统？”，我会毫不犹豫的回答 —— Samba！

Samba 是微软 CIFS	协议的开源实现，几乎所有智能路由器上的 USB 共享功能都是通过 Samba 实现的。

## 安装 Samba

在《[通过 SSH 远程管理服务器](initialization/use-ssh.md)》中的 `用主机名访问 NAS 主机` 部分已经安装了 Samba 软件包，如果没有安装：

```
getnas@getnas:~$ sudo apt install samba
```