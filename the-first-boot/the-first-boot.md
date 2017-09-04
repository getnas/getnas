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
* `~` 代表当前用户的家目录，即 `/home/getnas` 目录。

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

如果你的路由器没有 DHCP 服务器，可以参考以下格式设置静态 IP 地址：

```
......
auto enp2s0
allow-hotplug enp2s0
iface enp2s0 inet static
	address 192.168.10.100
	netmask 255.255.255.0
	gateway  192.168.10.1
```
`static` 代表该网卡设备使用静态 IP 地址，相关的设置为：

* `address` 静态 IP 地址
* `netmask` 子网掩码
* `gateway` 网关 IP 地址

## 配置无线网卡

Linux 系统下的无线网卡设置分两部分，第一步，安装无线网卡的芯片驱动程序，第二步设置无线网络。从 Debian 6 开始，所有闭源无线网卡驱动均被移除。

目前，可以免驱的无线网卡数量非常有限，其中 USB 无线网卡有 Realtek RTL8187B（802.11G）和 Atheros AR9170（802.11N），以及所有 Atheros 芯片的 Mini PCIe 网卡。

使用 `lsusb` 命令可以查看 USB 无线网卡的型号信息，如下所示，计算机连接了一个 `RTL8192CU` 芯片的 USB 无线网卡：

```
getnas@getnas:~$ lsusb
......
Bus 003 Device 003: ID 0bda:8178 Realtek Semiconductor Corp. RTL8192CU 802.11n WLAN Adapter
Bus 003 Device 002: ID 0930:6545 Toshiba Corp. Kingston DataTraveler 102/2.0 / HEMA Flash Drive 2 GB / PNY Attache 4GB Stick
```

如果你的计算安装了无线网卡，在执行 `ip a` 命令时会有列出类似下面的相关的设备信息：

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
3: wlxe84e0638b089: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether e8:4e:06:38:b0:89 brd ff:ff:ff:ff:ff:ff
```

执行 `iwconfig` 命令查看无线网卡状态：

```
getnas@getnas:~$ sudo iwconfig
lo        no wireless extensions.

wlxe84e0638b089  IEEE 802.11  ESSID:off/any
          Mode:Managed  Access Point: Not-Associated   Tx-Power=0 dBm
          Retry short limit:7   RTS thr=2347 B   Fragment thr:off
          Encryption key:off
          Power Management:on
```

通过查看上面的命令输出，可以知道无线网卡设备名为 `wlxe84e0638b089`，启用无线网卡：

```
getnas@getnas:~$ sudo ip link set wlxe84e0638b089 up
```

扫描可用的无线网络，当周边无线信号较多时会有很多结果输出，以下仅截取部分结果作为演示，其中 `ESSID` 为无线网络名，是最主要的识别信息：

```
getnas@getnas:~$ sudo iwlist scan
......
          Cell 04 - Address: F0:B4:29:D9:04:55
                    Channel:6
                    Frequency:2.437 GHz (Channel 6)
                    Quality=42/70  Signal level=-68 dBm
                    Encryption key:on
                    ESSID:"JIXIANG"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 9 Mb/s
                              18 Mb/s; 36 Mb/s; 54 Mb/s
                    Bit Rates:6 Mb/s; 12 Mb/s; 24 Mb/s; 48 Mb/s
......
```

目前，无线网络通畅会使用 `WPA-PSK` 或 `WPA2-PSK` 加密方式设置，使用 `wpa_passphrase` 命令生成 `psk` 值，如下：

```
getnas@getnas:~$ wpa_passphrase JIXIANG wifi-password
network={
	ssid="JIXIANG"
	#psk="wifi-password"
	psk=f0803bdb42ecadb659924c46d48f938d64ffb833dfbb517729b8f602d99b100w
}
```

使用 `nano` 编辑器打开 `/etc/network/interfaces` 网络接口配置文件：

```
getnas@getnas:~$ sudo nano /etc/network/interfaces
```

在文件末尾新增以下配置：

```
......
auto wlxe84e0638b089
iface wlxe84e0638b089 inet dhcp
        wpa-ssid JIXIANG
        wpa-psk f0803bdb42ecadb659924c46d48f938d64ffb833dfbb517729b8f602d99b100w
```

> 注意：请使用你实际查询到的无线网卡和无线网络相关信息替换上述内容。`wlxe84e0638b089` 替换为你的无线网卡设备名，`JIXIANG` 替换成你家或公司的无线网络名，`wpa-psk` 的值替换成你执行 `wpa_passphrase` 命令生成的 `psk` 值。

使用 `ifup` 命令启动无线网卡：

```
getnas@getnas:~$ sudo ifup wlxe84e0638b089
Internet Systems Consortium DHCP Client 4.3.5
Copyright 2004-2016 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/wlxe84e0638b089/e8:4e:06:38:b0:89
Sending on   LPF/wlxe84e0638b089/e8:4e:06:38:b0:89
Sending on   Socket/fallback
DHCPDISCOVER on wlxe84e0638b089 to 255.255.255.255 port 67 interval 5
DHCPDISCOVER on wlxe84e0638b089 to 255.255.255.255 port 67 interval 9
DHCPREQUEST of 192.168.10.135 on wlxe84e0638b089 to 255.255.255.255 port 67
DHCPOFFER of 192.168.10.135 from 192.168.10.1
DHCPACK of 192.168.10.135 from 192.168.10.1
bound to 192.168.10.135 -- renewal in 17615 seconds.
```

再次执行 `ip a` 命令，可以看到无线网卡已经获取到了 IP 地址：

```
getnas@getnas:~$ ip a
......
4: wlxe84e0638b089: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether e8:4e:06:38:b0:89 brd ff:ff:ff:ff:ff:ff
    inet 192.168.10.135/24 brd 192.168.10.255 scope global wlxe84e0638b089
       valid_lft forever preferred_lft forever
    inet6 fe80::ea4e:6ff:fe38:b089/64 scope link
       valid_lft forever preferred_lft forever
```

## 总结

NAS 服务器离不开网络，就像鱼儿离不开水。初次启动系统，首要的任务就是将计算机的网络连接设置妥当。

不难发现，配置简单是有线连接的天然优势，但它却摆脱不了网线的束缚。无线网卡虽然可以省去网线的凌乱，但配置起来确实麻烦。

由于无线网卡的芯片驱动几乎都是闭源的，因此在 Debian 这种严格发扬开源精神的发行版上使用的确要一番波折。

网络配置妥当，接下来让我们为操作系统做进一步的初始化工作吧。