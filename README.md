# 一台电脑的 NAS 之旅

本手册将指导您构建一台 `NAS` 网络存储服务器，您可以使用手边闲置的台式机、笔记本或像树莓派一样的卡片电脑。我们将从系统安装开始，一步一步打造拥有丰富功能的存储服务器。

## 构建目标

跟随本手册的指导，你可以将一台普通的计算机（PC）打造成具有以下功能的 NAS 服务器：

- 具有扩容能力的存储系统
- 多用户私有网盘
- 全能下载机
- 网络文件共享
- 跨平台文件同步

搭配云服务器、国际域名、邮件推送等云计算资源，还可以实现以下功能：

- 网盘远程访问
- 基于 P2P 的虚拟专用网
- E-mail 故障通知

## 技能要求

本手册的绝大多数操作需在 Linux 命令行界面进行，没有 Linux 系统使用基础的用户可能无法顺畅完成 NAS 服务器的构建。

腾讯云提供了免费的上机课程 [【Linux 基础入门】](https://cloud.tencent.com/developer/labs/lab/10000)，你可在45分钟内免费使用腾讯云服务器实际操练最常用的 Linux 命令。

## 风险提示

本手册尚在创作，在此期间，目录、标题和内容随时都可能发生较大变动。建议 **暂时** 不要构建用于生产环境的 NAS 服务器。

## 目录

- [概述](summary.md)
- [准备](preparations.md)
- [系统安装](os/README.md)
    - [制作系统安装盘](os/usb-installer.md)
	- [安装系统](os/installation.md)
	- [第一次启动系统](os/first-boot.md)
	- [使用 SSH 访问 NAS 服务器](os/use-ssh.md)
- [存储管理](storage/README.md)
	- [准备数据盘](storage/prepare-hdd.md)
	- [系统 U 盘分区容量调整](storage/resize-flashdrive.md)
		- [方案一 直接使用硬盘分区](storage/case-one.md)
		- [方案二 硬盘 + LVM](storage/case-two.md)
		- [方案三 直接使用 RAID 磁盘阵列](storage/case-three.md)
		- [方案四 RAID + LVM](storage/case-four.md)
- [Samba 文件共享](samba)
	- [Samba 共享配置](samba/create-samba-share.md)
	- [用各种设备访问共享](samba/access-samba-share.md)
- [Syncthing - 跨平台文件同步](syncthing)
- [下载工具](download-machine)
	- [全能下载工具 Aria2 配置](download-machine/aria2-settings.md)
	- [BT/PT 下载工具 Transmission 配置](download-machine/transmission-settings.md)
	- [BT 下载工具 qBittorrent 配置](download-machine/qbittorrent-settings.md)
- [NextCloud - 多用户私有云网盘](nextcloud)
   - [使用 Let's Encrypt 实现加密的 HTTPS 协议](nextcloud/frp_letsencrypt.md) 
- [Apple Time Machine](time-machine)
	- [Time Machine 配置](time-machine/time-machine-settings.md)
	- [使用 Time Machine](time-machine/time-machine-usage.md)
- [常见问题](questions)
	- [UEFI 模式 Debian 启动失败进入 initramfs](questions/uefi-cannot-boot.md)
	- [修改 Docker 默认存储目录](questions/docker-root.md)

## 采用的开源项目

- [Ubuntu](https://www.ubuntu.com/server)
- [NextCloud](https://www.nextcloud.com)
- [Syncthing](https://syncthing.net/)
- [Aria2](https://aria2.github.io/)
- [qBittorrent](https://www.qbittorrent.org/)
- [Samba](https://www.samba.org/)
- [Apache](http://httpd.apache.org/)
- [MariaDB](https://downloads.mariadb.org/)
- [Tinc](https://www.tinc-vpn.org/)
- [Netatalk](http://netatalk.sourceforge.net/)
- [Avahi](http://avahi.org/)
- [frp](https://github.com/fatedier/frp)
- [webui-aria2](https://github.com/ziahamza/webui-aria2)
- [Docker](https://www.docker.com/)
- [Transmission](https://transmissionbt.com)
- [Etcher](https://etcher.io/)

## 写作规范

- [项目写作规范](writing-guidelines.md)

## 赞助 & 合作

- [提供赞助](sponsor/sponsor.md)

## 版权 & 许可

版权所有 ©️ 2017-2018 [于鸿儒](https://twitter.com/herald_yu) (Herald Yu)

[<img alt="知识共享许可协议" style="border-width:0" src="images/by-nc-nd-88x31.png">](http://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh)

本作品采用 [署名-非商业性使用-禁止演绎 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 进行许可。
