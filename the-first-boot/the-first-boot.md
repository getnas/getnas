# 第一次启动

经过了前面的操作，Debian 操作系统已经安装到了我们的 U 盘中，现在就可以从这个 U 盘启动计算机，开始我们的 NAS 构建之旅了。

为了避免硬盘中存在引导信息干扰 U 盘系统启动，建议初次启动计算机时，断开所有硬盘，只保留安装了 Debian 系统的 U 盘。当系统启动成功后，再将硬盘接入计算机。（如果你准备的是全新的或完全格式化过的硬盘，则无需断开。）

## 登录系统

看到下图所示的界面，就表示 Debian 系统已经成功启动了。在 `login:` 提示符处键入你在系统安装时创建的用户名，按 `Enter` 回车键确认。当出现 `Password:` 时输入你设置的密码，按 `Enter` 回车键确认。

![系统登录](system-login.png)

如果用户名和密码输入错误，系统会提示你重新输入。

身份验证通过，就能看到类似下面的欢迎信息。

```
Linux getnas 4.9.0-3-amd64 #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.

getnas@getnas:~$
```

当你看到最后一行提示符为 `$` 或 `#`，则表明你已经成功登录到 Linux 系统。

> 命令提示符 `getnas@getnas:~$` 中，第一个 `getnas` 为当前登录的用户名，`@` 为分隔符，第二个 `getnas` 为主机名，`:` 也是分隔符，`~` 为用户当前所在的位置。`$` 表明当前身份为普通用户。

* `$` 代表当前的用户身份为普通用户。
* `#` 代表当前的用户身份为超级管理员。

## 配置有线网卡

如果你是参照我们前文提供的方法安装的 Debian 系统，那么在系统初次启动时网络会被自动配置妥当无需额外设置。

如果你购买了 GetNAS 商店的 Debian 系统 U 盘或是你要从另外一台计算机启动 U 盘系统，那么初次启动计算机以后需要手动配置有线网络，只需配置一次。

### 第一步 查看网卡设备名

输入 `ip a`，按 `enter` 回车键执行命令。你会看到类似下面的代码，共有两个设备，一个名称为 `lo` 为主机内部的循环设备，另一个名为 `enp2s0` 是我们要配置的网卡。

```
getnas@getnas:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp2s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b8:97:5a:6c:ba:87 brd ff:ff:ff:ff:ff:ff
```

> 提示：这里的网卡设备名为 `enp2s0`，你查到的名称可能与此不同，请务必使用你查询到的真实网卡名称进行网络设置。

### 第二步 配置网卡

Debian 系统的网卡信息在 `/etc/network/interfaces` 文件中配置，使用 `nano` 编辑器打开。注意，命令前面要加上 `sudo` 命令，这样才能编辑系统配置文件，系统可能会要求你输入用户密码。

```
getnas@getnas:~$ sudo nano /etc/network/interfaces
```

可以看到类似下面的信息：

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp
```

在最后两行代码前添加一行 `auto enp2s0`，将最后两行中的 `enp0s3` 替换为 `enp2s0`，修改后的代码应类似下面这样：

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp2s0
allow-hotplug enp2s0
iface enp2s0 inet dhcp
```

> 注意：请使用你在第一步中实际查询到的网卡名称替换配置文件中的 `enp2s0`。

编辑完成后，使用组合键 `CTRL + o` 保存文件，`CTRL + x` 退出 `nono` 编辑器。

经过如上设置，名称为 `enp2s0` 的网卡会随计算机开机而启动，并自动从路由器的 DHCP 服务器获取动态 IP 地址。

## 配置无线网卡

## 系统更新

## 实现局域网主机名访问

## 配置数据盘