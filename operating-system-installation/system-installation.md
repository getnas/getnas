# 安装 Debian 操作系统

### 引言

本文将指导你使用此前制作的 U 盘系统安装盘，将 Debian 操作系统安装到另外的那个 U 盘上。

### 先决条件

使用 U 盘系统盘安装 Linux 操作系统，要求你必须了解如何从 USB 设备引导计算机启动。通常，大多数计算机设计为在开机时按下 `F12` 键呼出引导选项菜单。也有可能是 `F8`、`F9`，详情请参照你的主板说明书。

## 系统安装

### 第一步 从 U 盘安装盘引导启动计算机

将制作好的 U 盘系统安装盘和准备的另一个 8GB 以上容量的空白 U 盘插入用于构建 NAS 的计算机。

开机并设置从 U 盘安装盘引导启动计算机，启动成功后会看到 Debian 系统安装菜单，如下图。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-method.png" alt="debian 安装方式选择">

选择菜单中的第一项 `Graphical install`（图形界面安装），按回车确认。这种安装模式支持同时使用键盘和鼠标进行操控。

### 第二步 选择安装语言

点选列表中的 `Chinese (Simplified) - 中文(简体)`，点击 `Continue` 按钮继续。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-language.png" alt="选择安装语言">

紧接着会提示安装过程的中文语言包可能不完整，是否使用所选语言继续安装，如下图，点选 `是`，点击 `Continue` 按钮继续。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-language2.png" alt="确认安装语言">

### 第三步 本地化相关设置

提示选择所在区域，界面对这个步骤提供了明确的介绍，如下图，在列表中点选 `中国`，然后点击 `继续` 按钮。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-area.png" alt="选择所在区域">

提示选择键盘，默认已选中 `汉语`，点击 `继续` 按钮即可。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-area.png" alt="配置键盘">

紧接着系统安装器会自动检测镜像，自动配置网络等一系列自动化配置，无需我们干预，等待完成即可。

### 第四步 设置主机名和域名

主机名建议设置一个简单好记的英文单词或字母组合，因为以后需要用主机名和域名访问这台 NAS 服务器，如果你不知道设置什么好，那就设置成 `getnas` 吧。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-hostname.png" alt="设置主机名">

这里建议将域名设置为 `local`，这样一来，以后在局域网中就可以用 `getnas.local` 地址访问这台 NAS 服务器了。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-domain.png" alt="设置域">
