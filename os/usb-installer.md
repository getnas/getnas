# 制作系统安装盘

曾经我们通过将系统镜像刻录到光盘制作`系统安装光盘`，随着光驱逐渐被淘汰，大家现在广泛改用 U 盘来制作系统安装盘。

## 下载系统 ISO 镜像

在本指南使用 `Ubuntu 16.04 Server (LTS)` 操作系统，即专门针对服务器环境优化的版本。

**下载地址：**[https://www.ubuntu.com/download/alternative-downloads](https://www.ubuntu.com/download/alternative-downloads)

> 提示：Ubuntu 18.04 是最新发布的 LTS 版本，但许多应用暂未发布适配的软件包，因此我们暂时不用这个版本。

> **LTS**: Long Term Support - 长期支持，Ubuntu LTS 版本拥有长达 5 年的更新支持。

### 架构选择

下载 ubuntu 系统镜像时会看到镜像文件名中会包含 `amd64` 或 `i386` 等架构名称，该如何选择呢？

`amd64` 代表 64 位版本的系统，`i386` 代表
32 位版本的系统。

如果你的计算机是近几年购买的，几乎都是支持 `64位` 的。如果硬件支持，NAS 服务器应选用 `64位` 的系统。

如果你不确定自己的硬件是否支持 64 位系统，网上搜索 CPU 型号即可了解相关信息。

## 使用 Etcher

`Etcher` 是一款开源的跨平台操作系统镜像烧录工具，用它制作系统安装 U 盘非常简单。

> **官网下载**：https://etcher.io/

Etcher 支持 Linux、Windwos 以及 MacOS 操作系统，请根据你当前使用的计算机系统进行选择下载。

![Etcher](img/etcher.png)

Etcher 的使用非常简单，运行程序，选择已经下载的系统 `iso` 镜像，然后选择 U 盘，最后点击 `Flash` 按钮等待烧录完成即可。

## 其他工具

与 Etcher 类似的工具还有：

- [unetbootin](https://unetbootin.github.io/)
- [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/)