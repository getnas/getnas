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