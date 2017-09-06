# SSH 远程访问服务器

SSH - Secure (安全) Shell，是专门用作远程的登录的可靠协议。

安装 Debian 系统时 `选择预装软件` 的环节我们特别勾选了 `SSH Server`。因此，我们的 NAS 主机默认就已开启了 SSH 协议支持。

## 通过 SSH 远程登录 NAS 主机

绝大多数的 Linux 发行版和 Mac OS X 系统默认预装了 SSH 客户端，因此可在 `终端` 直接使用 `ssh` 命令。

Microsoft Windows 系统没有预置，因此需要用户自行下载安装 SSH 客户端，推荐使用开源的 [Putty](http://www.putty.org/)。

### Linux / Unix / MacOS

### Microsoft Windows

#### 第一步 下载 Putty

请根据你的操作系统架构选择下载 32 位或 64 位的 Putty：

* [Putty 32 位](https://the.earth.li/~sgtatham/putty/latest/w32/putty.exe) 
* [Putty 64 位](https://the.earth.li/~sgtatham/putty/latest/w64/putty.exe)

## 用主机名访问 NAS 主机

在安装系统时我们设置了主机名 `getnas` 和域名 `local`，而且还强调将来要以 `getnas.local` 的域名形式在局域网中访问这台 NAS 主机。

想要实现这种域名访问，需要安装 `avahi-daemon` 软件包。它主要用作局域网的设备发现和广播本机的主机名，这样就可以实现其他设备通过主机名访问到本机。

安装 `avahi-daemon`：

```
getnas@getnas:~$ sudo apt install avahi-daemon
```

现在，使用局域网中任何一台 Linux 计算机运行 `ping getnas.local` 命令即可看到成功连通的输出。

由于 Windows 系统目前还不能识别 NAS 的主机名，因此进一步的测试我们等到配置 Samba 共享时再做介绍。

