# 操作系统安装

### 引言

GetNAS 项目采用开源的 Linux 发行版 Debian 构建，本文内容将指导你创建 U 盘系统安装盘，并将操作系统安装到 U 盘上。

## 先决条件

在阅读后面内容前，请确保你已经阅读了 [准备工作](preparation.md)，并遵照文中要求准备了相关物料。

## 下载 Deiban 系统镜像

Debian 官方下载频道：[https://www.debian.org/CD/](https://www.debian.org/CD/)

如果你所准备的将用作构建 NAS 服务器的计算机使用的是 Intel 或 AMD 中央处理器（CPU），根据对 64 位支持情况选择以下两个版本之一下载。`i386` 对为 32 位系统，`amd64` 为 64 位系统。

Debian amd64 最新版：[https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/](https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/)

Debian i386 最新版：[https://cdimage.debian.org/debian-cd/current/i386/iso-cd/](https://cdimage.debian.org/debian-cd/current/i386/iso-cd/)

> **注意**：`.iso` 后缀即操作系统镜像文件，由于我们要构建的是没有桌面环境的服务器，因此，请下载文件大小为 200 MB 左右的 `netinst` 版本，例如，debian-9.1.0-amd64-netinst.iso。


## 制作 U 盘安装盘

制作 U 盘安装盘，即把下载的 ISO 系统镜像文件写入到 U 盘的过程。不同的操作系统，有不同的制作方法，下面分别介绍在 Windows、Linux 以及 MAC OS X 系统下的制作方法：

### Linux

**第一步 确认 U 盘设备名**

U 盘插入后，它将被映射到名为 `/dev/sdX` 的设备，名称中的 `X` 是 a-z 的字母。打开终端（Terminal），使用 `fdisk -l` 命令查看 U 盘设备名：

```
$ sudo fdisk -l
......
Disk /dev/sdc: 14.8 GiB, 15822487552 bytes, 30903296 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: B94773B3-8256-4066-BB26-884DBF50670C
......
```

上述命令会打印出所有接入到计算机上的存储设备，为突出重点省略了其他硬盘信息，仅保留 U 盘信息。`/dev/sdc` 即 U 盘的设备名，`14.8 GiB` 为 U 盘容量。

> **特别注意**：你的设备名可能不是 `sdc`，该名称是由系统根据设备的接入顺序自动分配的，分配顺序一般为 `sda`、`sdb`、`sdc` 以此类推，你必须依据实际的命令输出来确定设备名。

**第二步 将系统镜像写入 U 盘**

为避免新手读者产生混淆，我们进入系统镜像所在的目录执行后续操作，假设系统镜像下载到了 `~/Downloads` 目录，镜像名为 `debian-9.1.0-amd64-netinst.iso`。

```
$ cd ~/Downloads
```
> **提示**：在 Linux 系统下，`~` 符号代表当前用户的 `home` 目录，假设当前用户名为 `herald`，那么 `~/Downloads` 目录的实际路径为 `/home/herald/Downloads`，使用 `~` 可以少输入很多字母。

使用 `dd` 命令将镜像写入 U 盘：

```
~/Downloads$ sudo cp ./debian-9.1.0-amd64-netinst.iso /dev/sdc
```

> 注意：最后的设备名 `/dev/sdc` 要替换成你实际查到的设备名。

镜像写入需要一些时间，再次看到终端提示符 `$` 就代表镜像写入完成了，紧接着执行 `sync` 同步命令：

```
$ sudo sync
```

这样 Debian 系统安装盘就制作完成了。

### Windows



### UNetbootin

UNetbootin 是一款实用的小工具，能够快速制作各种主流 Linux 发型版的系统安装 U 盘。但在我们不推荐使用，原因是该软件会在执行过程中篡改系统镜像，存在一定的安全风险。