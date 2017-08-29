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

**所在区域**：界面对这个步骤提供了明确的介绍，如下图，在列表中点选 `中国`，然后点击 `继续` 按钮。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-area.png" alt="选择所在区域">

**键盘**：默认已选中 `汉语`，点击 `继续` 按钮即可。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-keymap.png" alt="配置键盘">

### 第四步 设置主机名和域名

**主机名**：建议设置一个简单好记的英文单词或字母组合，因为以后需要用主机名和域名访问这台 NAS 服务器，如果你不知道设置什么好，那就设置成 `getnas` 吧。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-hostname.png" alt="设置主机名">

**域名**：建议将设置为 `local`，这样一来，以后在局域网中就可以用 `getnas.local` 地址访问这台 NAS 服务器了。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-domain.png" alt="设置域">

### 第五步 用户和密码

**超级管理员 (root)**：请认真阅读屏幕上的提示信息，出于系统安装考虑，通常不设置 `root` 密码，即不直接使用超级管理员管理系统，而是创建一个普通用户并将其加入到管理组，当需要管理员权限时，通过在命令前添加 `sudo` 或通过使用 `sudo su` 切换到 root 用户身份。

因此，这一步不做设置，点击 `继续` 按钮跳过。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-root-password.png" alt="root 密码">

接下来，安装器会引导我们创建一个新的普通用户。

**用户全名**：仅作识别之用，字母大小写均可。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-user-fullname.png" alt="用户全名">

**用户名**：安装器默认将设置的用户全名变成小写设为用户名，你可以按需修改。建议设置短小易记的名称，今后通过 SSH 连接服务器时需要频繁使用这个用户名。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-username.png" alt="用户名">

**密码**：为当前正在创建的新用户创建一个密码，密码长度和复杂度没有限制，但建议你设置一个足够复杂的密码。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-userpasswd.png" alt="用户密码">

### 第六步 磁盘分区

由于我们要将 Debian 操作系统安装到 U 盘上，无需复杂设置，直接选择 `向导 - 使用整个磁盘` 这一项即可。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-partition.png" alt="磁盘分区">

**选择磁盘**：注意不要选错，一定要选择 U 盘，通过设备名称和容量很容易判断。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-select-disk.png" alt="选择磁盘">

**选择分区方案**：选择 `将所有文件放在同一个分区中（推荐新手使用）` 这一项。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-partition-plan.png" alt="分区方案">

**调整分区**：安装器已经帮助我们自动创建了分区，一个挂载到 `/` 根目录的 ext4 分区，一个 `swap` 交换空间分区，如下图。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-partition-result.png" alt="调整分区">

> `Swap 交换空间` 类似于 Windows 中的 `分页文件`，主要作用是当计算机内存不足时，使用硬盘模拟内存，从而避免系统因资源不足导致的无响应或瘫痪等问题。

如今，内存容量越来越大，价格也越来越便宜，计算机很少会出现内存不足的问题。同时，受闪存设备的结构特点制约，在 U 盘上创建 `swap` 分区不但对系统运行速度提升没有任何帮助，还会加速 U 盘损坏。因此，强烈建议删除 `swap` 分区。

**删除 Swap 分区**：

鼠标双击 `swap` 分区那一行记录，如下图，在 swap 分区详情页中点击 `删除此分区`。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-delete-swap.png" alt="删除 swap 分区">

最终，你会看到类似下图所示的结果。删除了 swap 分区，U 盘上空闲出了一部分容量，这里我们暂时不用管它。我们会在系统安装完成后，使用 `resize2fs` 命令扩展 `/` 分区容量到最大。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-partition-final.png" alt="分区最终结果">

由于我们删除了 swap 分区，安装器会给出提示，如下图，选择 `否` 忽略即可。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-partition-alert.png" alt="swap 警告">

**操作确认**：选择 `是`，安装器会按照上面的设定，在 U 盘上创建并格式化分区。

<img src="https://raw.githubusercontent.com/getnas/getnas/master/operating-system-installation/debian-installation-partition-confirm.png" alt="分区确认">