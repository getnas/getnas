# 第一次启动系统

## 用户登录

初次启动系统，会看到下图所示的登录界面。

![](img/os_1.png)

在 `GetNAS login:` 提示符下输入用户名 `getnas` 按 `回车` 确认，在 `Password:` 提示符下输入密码。

登录成功后会看到 `getnas@GetNAS:~$` 提示符。

![](img/os_2.png)

## 系统更新

初次启动系统，有必要更新一下系统中旧的软件包。

**刷新软件列表**
```shell
getnas@GetNAS:~$ sudo apt update
```
> 执行命令时会要求输入密码

**更新软件包**
```shell
getnas@GetNAS:~$ sudo apt upgrade
```

> 执行更新命令时会有确认提示 `[Y/n]`，支持按 `回车` 键确认即可。

## 配置主机名访问

现在，如果我们要访问 NAS 服务器，必须通过局域网 IP 才行，然而主机的 IP 地址默认通过路由器的 `DHCP` 服务器分配，可能过一段时间就会发生变化。

如果我们为 NAS 服务器开启主机名访问功能，那么无论 IP 地址如何变化，都不会有影响了。

为了开启主机名访问功能，需要安装 `avahi-deamon` 和 `samba`。

```shell
getnas@GetNAS:~$ sudo apt install -y avahi-deamon samba
```

## 修改更快的软件源

如果你更新软件包的速度较慢，可以用国内的软件源镜像替换官方软件源，这里采用阿里云的镜像举例：

### 备份配置文件

Ubuntu 系统的软件源信息位于 `/etc/apt/sources.list` 文件中，为了防止不当修改产生不可预知的后果，在修改之前应该对其进行备份。

```shell
~$ sudo cp /etc/apt/sources.list /etc/apt/sources.list_bak
```

### 方案一 手动修改软件源

使用 `nano` 编辑器打开配置文件：

```shell
~$ sudo nano /etc/apt/sources.list
```

将其中的 `cn.archive.ubuntu.com` 和 `security.ubuntu.com` 均替换成 `mirrors.aliyun.com`。

> nano 是 Linux 系统下非常简单易用的一款编辑器，如果你初次使用，可以到网上搜索相关指南。

### 方案二 下载配置文件

如果你不想手动修改配置文件，可以在备份配置文件后，执行以下命令下载已经修改好的配置文件。

```shell
~$ sudo wget http://theurl.com/sources.list -P /etc/apt
```