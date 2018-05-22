# 制作系统安装盘

安装 Ubuntu 系统理论上只有三个步骤，首先下载系统安装镜像，然后将镜像写入 U 盘（或烧录到 CD），最后使用制作好的安装盘引导启动计算机完成系统的安装。

## 下载系统 ISO 镜像

在本指南中，我们要使用 `Ubuntu Server` 操作系统，即专门针对服务器环境设计的系统版本。

本指南使用 `Ubuntu 16.04 Server (LTS)`

**下载地址：**[https://www.ubuntu.com/download/alternative-downloads](https://www.ubuntu.com/download/alternative-downloads)

> 提示：Ubuntu 18.04 是最新发布的 LTS 版本，但很多应用尚未支持该版本，因此本指南暂时不会选用。

> **LTS**: Long Term Support - 长期支持，Ubuntu LTS 版本拥有长达 5 年的更新支持。

### 架构选择

下载 ubuntu 系统镜像时会看到镜像文件名中会包含 `amd64` 或 `i386` 等架构名称，该如何选择呢？

`amd64` 代表 64 位版本的系统，`i386` 代表
32 位版本的系统。

如果你的计算机是近几年购买的，几乎都是支持 `64位` 的。对于 NAS 服务器构建，我们总是应该在支持的前提下有限使用 `64位` 的系统。

如果你不确定系统是否支持 64 位，可以搜索一下 CPU 型号即可了解相关信息。

## 使用 Etcher

`Etcher` 是一款开源的跨平台操作系统镜像烧录工具，可以非常简单的制作系统安装 U 盘。

> **官网下载**：https://etcher.io/

Etcher 支持 Linux、Windwos 以及 MacOS 操作系统，请根据你当前使用的计算机系统进行选择下载。

![Etcher](img/etcher.png)

Etcher 的使用非常简单，运行程序后，首先选择已经下载好的系统 `iso` 镜像，然后选择 U 盘，最后点击 `Flash` 按钮等待烧录完成即可。