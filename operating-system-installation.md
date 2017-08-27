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