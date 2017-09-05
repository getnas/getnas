# 通过 SSH 远程管理服务器

## 用主机名访问 NAS 主机

在安装系统时我们设置了主机名 `getnas` 和域名 `local`，而且还强调将来要以 `getnas.local` 的域名形式在局域网中访问这台 NAS 主机。

想要实现这种域名访问，需要安装 `avahi-daemon` 软件包。它主要用作局域网的设备发现和广播本机的主机名，这样就可以实现其他设备通过主机名访问到本机。

安装 `avahi-daemon`：

```
getnas@getnas:~$ sudo apt install avahi-daemon
```

现在，使用局域网中任何一台 Linux 计算机运行 `ping getnas.local` 命令即可看到成功连通的输出。

由于 Windows 系统目前还不能识别 NAS 的主机名，因此进一步的测试我们等到配置 Samba 共享时再做介绍。

## 通过 SSH 远程登录 NAS 主机