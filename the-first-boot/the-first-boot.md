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
Last login: Fri Sep  1 17:25:51 2017 from 192.168.10.193
getnas@getnas:~$
```

通常，当你看到最后一行提示符为 `$` 或 `#`，则表明你已经成功登录到 Linux 系统。

* `$` 代表当前的用户身份为普通用户。
* `#` 代表当前的用户身份为超级管理员。

## 配置有线网卡

## 配置无线网卡

## 系统更新

## 实现局域网主机名访问

## 配置数据盘