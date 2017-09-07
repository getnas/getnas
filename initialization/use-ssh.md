# SSH 远程访问服务器

SSH - Secure (安全) Shell，是一种用于远程服务器登录和执行命令的加密网络协议。

安装 Debian 系统时，之所以 `选择预装软件` 环节勾选 `SSH Server`，目的就是为开启 NAS 主机的 SSH 协议支持。

## SSH 连接

### NAS 服务器的 IP 地址

在前面已经介绍过 Debian 网络配置相关内容，查询 NAS 主机的 IP 地址方法有很多种，最简单直接的办法就是在 NAS 服务器上执行 `ip a` 命令：

```
getnas@getnas:~$ ip a
......
2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether b8:97:5a:6c:e0:92 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.102/24 brd 192.168.10.255 scope global enp2s0
       valid_lft forever preferred_lft forever
    inet6 fe80::ba97:5aff:fe6c:e092/64 scope link
       valid_lft forever preferred_lft forever
```

可以看到 `192.168.10.102` 就是这台 NAS 服务器的 IP 地址。我们以此地址为例介绍其他电脑如何使用 SSH 客户端登录这台 NAS 服务器。

### Linux / Unix / MacOS

绝大多数的 Linux 发行版和 Mac OS X 系统默认预装 SSH 客户端，可在 `终端` 直接使用 `ssh` 命令。

#### 第一步 打开终端

Mac OS X 系统在 `LaunchPad` -> `其他` 文件夹中可以找到终端。

Ubuntu 桌面版用 `CTRL + ATL + T` 组合快捷键可直接打开终端。

其他 Unix-like 系统请在系统菜单的附件目录中查找。

#### 第二步 启动连接

在终端中使用 `ssh` 命令发起服务器连接请求：

```
$ ssh getnas@192.168.10.102
```

* `getnas` 为 NAS 服务器中的用户名（系统安装时创建）
* `@` 为连接符或分隔符
* `192.168.10.102` 是 NAS 服务器在局域网中的 IP 地址

> 注意：请使用你的 NAS 服务器上实际的信息替换上述命令中的 `用户名` 和 `IP` 地址。

初次通过 SSH 登录 NAS 服务器，终端会发出类似下面的提示，即询问我们是否确认要与 NAS 服务器建立连接：

```
he authenticity of host '192.168.10.102 (192.168.10.102)' can't be established.
ECDSA key fingerprint is SHA256:pYF93FVqmh5uwCTv8LL3Q8n/kHzEHmGRFwUOidbqW5s.
Are you sure you want to continue connecting (yes/no)?
```

这时我们要手动输入 `yes` 并按 `Enter` 回车键确认，若输入 `no` 为放弃连接。

接着，终端再次发出提示，要求输入用户密码：

```
getnas@192.168.10.102's password:
```

输入密码，按 `Enter` 回车键确认，连续三次输错密码会自动退出 `ssh` 命令。

> 提示：在终端中输入密码时没有任何提示，即在我们输入密码时终端提示符不会有任何反应，这是一种非常好的安全策略。

密码验证无误，即可成功登录 NAS 服务器：

```
Linux getnas 4.9.0-3-amd64 #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Sep  7 09:31:28 2017 from 192.168.10.193
getnas@getnas:~$
```

### Microsoft Windows

Microsoft Windows 系统默认没有对 SSH 协议提供支持，因此需要用户自行下载安装 SSH 客户端，推荐使用开源的 [Putty](http://www.putty.org/)。

#### 第一步 下载 Putty

请根据你的操作系统架构选择下载 32 位或 64 位的 Putty：

* [Putty 32 位](https://the.earth.li/~sgtatham/putty/latest/w32/putty.exe) 
* [Putty 64 位](https://the.earth.li/~sgtatham/putty/latest/w64/putty.exe)

#### 第二步 连接配置

下载的 `putty.exe` 是绿色文件，无需安装双击即可直接运行，程序界面如下图：

![Putty 主界面](putty.png)

我们只需把 NAS 服务器的 IP 地址输入到 `Host Name` 文本框中，然后点击 `Open` 按钮即可。

初次连接会弹出下图所示的安全提示窗口，点击 `是` 确认即可。

![SSH 安全提示](putty-security-alert.png)

#### 第三步 登录 NAS 服务器

当 Putty 提示 `login as:` 时输入我们在 NAS 服务器上创建的用户名，提示 `getnas@192.168.10.102‘s password:` 时输入用户密码。

用户名和密码验证通过，即可成功登录 NAS 服务器，如下图：

![](putty-login.png)

## 用主机名访问 NAS 主机

在安装系统时我们设置了主机名 `getnas` 和域名 `local`，并且强调要以 `getnas.local` 的域名形式在局域网中访问这台 NAS 主机。

在局域网中使用域名访问 NAS 服务的主要优点在于无需记忆 IP 地址，这对使用 DHCP 获取动态 IP 的 NAS 服务器来说是一种福音。

想要实现这种域名访问，需要 `avahi-daemon` 软件包的主机名广播功能，同时还需要 `samba` 的 NetBIOS 功能。

### 安装 `Avahi` 和 `Samba`

```
getnas@getnas:~$ sudo apt install avahi-daemon samba
```

现在，使用局域网中任何一台 Unix-like 系统的计算机执行 `ping getnas.local` 命令即可看到成功连通的输出。

Microsoft Windows 系统的计算机，在 `命令提示符` 中执行 `ping getnas` 命令即可看到相似的连通输出。

> 注意：Windows 系统是借助 Samba 的 NetBIOS 功能实现对 NAS 服务器主机名的访问，地址中不要使用 `.local` 域名后缀。

### SSH 登录时使用主机名

**Linux/Unix/MacOS** 使用地址 `getnas.local`：

```
macbook:~ Herald$ ssh getnas@getnas.local
The authenticity of host 'getnas.local (192.168.10.102)' can't be established.
ECDSA key fingerprint is SHA256:pYF93FVqmh5uwCTv8LL3Q8n/kHzEHmGRFwUOidbqW5s.
Are you sure you want to continue connecting (yes/no)?
```

**Microsoft Windows** 使用主机名 `getnas`：

只需在 Putty 的 `Host Name` 中输入 `getnas` 即可，第一次使用主机名连接到 NAS 服务器，客户端仍然会发出安全提示，所有操作与使用 IP 地址连接时完全相同。

## 总结

不只是其他计算机，局域网中所有能够使用 SSH 客户端的手机等设备都可以建立与 NAS 远程连接。至此，NAS 服务器已经不再需要显示器和键盘了。
