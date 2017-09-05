# 系统初始化

网络连接配置妥当后，接下来要做一些最基础的系统环境配置。

## 系统更新

一般而言，系统更新就是将系统中软件包升级到最新版本的过程。我们需要定期从远程镜像服务器下载最新的软件包列表，它就像目录一样，帮我们识别哪些软件包有新版本，然后使用命令，更新软件包到最新版即可。

Debian 系统用 `apt` 命令管理软件包，我们主要用它：

* 安装软件
* 卸载软件
* 更新软件包

搭配 `update` 选项，从远程镜像服务器获取最新的软件包的列表：

```
getnas@getnas:~$ sudo apt update
[sudo] getnas 的密码：
获取:1 http://security.debian.org/debian-security stretch/updates InRelease [62.9 kB]
获取:2 http://security.debian.org/debian-security stretch/updates/main Sources [62.9 kB]
获取:3 http://security.debian.org/debian-security stretch/updates/main amd64 Packages [161 kB]
获取:4 http://security.debian.org/debian-security stretch/updates/main Translation-en [74.4 kB]
忽略:5 http://mirrors.163.com/debian stretch InRelease
获取:6 http://mirrors.163.com/debian stretch-updates InRelease [88.5 kB]
命中:7 http://mirrors.163.com/debian stretch Release
已下载 450 kB，耗时 12秒 (36.4 kB/s)
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
所有软件包均为最新。
```

搭配 `upgrade` 选项，执行软件包更新：

```
getnas@getnas:~$ sudo apt upgrade
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
正在计算更新... 完成
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 0 个软件包未被升级。
```

在执行 `apt upgrade` 命令时，如果存在可以更新的软件包，会有确认提示 `[Y/n]`，输入 `y` 或直接按 `Enter` 回车键执行更新，输入 `n` 不更新。

## 设置最快的镜像服务器

前面提到了更新 Debian 要从远程镜像服务器获取最新的软件包列表，而且软件包也要从镜像服务器下载，因此，访问镜像服务器的速度越快越好。

Debian 的镜像服务器遍及全球，如何选择一个与我们物理距离较近，且访问速度最快的服务器呢？

`netselect-apt` 这款工具能够在全球几百个 Debian 镜像服务器中，筛选出我们访问速度最快的 10 个，并帮我们把连接速度最快的服务器写入配置文件。

### 第一步 安装 netselect-apt

`apt` 命令搭配 `search` 选项可以搜索软件包：

```
getnas@getnas:~$ sudo apt search netselect-apt
[sudo] getnas 的密码：
正在排序... 完成
全文搜索... 完成
netselect-apt/stable 0.3.ds1-28 all
  speed tester for choosing a fast Debian mirror
```

`apt` 命令搭配 `install` 选项用以安装软件包：

```
getnas@getnas:~$ sudo apt install netselect-apt
正在读取软件包列表... 完成
正在分析软件包的依赖关系树
正在读取状态信息... 完成
将会同时安装下列软件：
  curl libcurl3 netselect
建议安装：
  dpkg-dev
下列【新】软件包将被安装：
  curl libcurl3 netselect netselect-apt
升级了 0 个软件包，新安装了 4 个软件包，要卸载 0 个软件包，有 0 个软件包未被升级。
需要下载 568 kB 的归档。
解压缩后会消耗 1,102 kB 的额外空间。
您希望继续执行吗？ [Y/n]
......
```

看到 `[Y/n]` 提示，直接按 `Enter` 回车键或输入 `y` 执行安装，输入 `n` 取消安装。

### 第二步 查询最快的镜像服务器

执行命令 `netselect-apt`，查询需要几分钟时间，运行完毕后会看到类似下面的结果：

```
getnas@getnas:~$ sudo netselect-apt
Using distribution stable.
Retrieving the list of mirrors from www.debian.org...
..........................................
The fastest 10 servers seem to be:

	http://mirror.xtom.com.hk/debian/
	http://mirrors.tuna.tsinghua.edu.cn/debian/
	http://opensource.nchc.org.tw/debian/
	http://ftp.jp.debian.org/debian/
	http://ftp.cn.debian.org/debian/
	http://mirror.rise.ph/debian/
	http://debian-mirror.sakura.ne.jp/debian/
	http://ftp.jaist.ac.jp/debian/
	http://debian.superhosting.cz/debian/
	http://ftp.nara.wide.ad.jp/debian/

Of the hosts tested we choose the fastest valid for HTTP:
        http://mirror.xtom.com.hk/debian/

Writing sources.list.
Done.
```

使用 `ls` 命令查看当前目录，可以看到有一个 `sources.list` 文件，它就是 `netselect-apt` 生成的连接到最快镜像服务器的配置文件。

```
getnas@getnas:~$ ls
sources.list
```

使用 `cat` 命令可将 `sources.list` 配置文件的内容输出到屏幕：

```
getnas@getnas:~$ cat sources.list
# Debian packages for stable
deb http://mirror.xtom.com.hk/debian/ stable main contrib
# Uncomment the deb-src line if you want 'apt-get source'
# to work with most packages.
# deb-src http://mirror.xtom.com.hk/debian/ stable main contrib

# Security updates for stable
deb http://security.debian.org/ stable/updates main contrib
```

### 第三步 使用新生成的镜像服务器

备份原配置文件：

```
getnas@getnas:~$ sudo cp /etc/apt/sources.list /etc/apt/sources.list-bak
```

用新配置文件替换原配置文件：

```
getnas@getnas:~$ sudo mv sources.list /etc/apt/sources.list
```

从新镜像服务器更新软件包列表：

```
getnas@getnas:~$ sudo apt update
```

每当你感觉到更新软件包速度不理想时，就可以按照上面的三个步骤重新查询并改用最快的镜像服务器。

## 用主机名访问 NAS 主机

在安装系统时我们设置了主机名 `getnas` 和域名 `local`，而且还强调将来要以 `getnas.local` 的域名形式在局域网中访问这台 NAS 主机。

想要实现这种域名访问，需要安装 `avahi-daemon` 软件包。它主要用作局域网的设备发现和广播本机的主机名，这样就可以实现其他设备通过主机名访问到本机。

安装 `avahi-daemon`：

```
getnas@getnas:~$ sudo apt install avahi-daemon
```

现在，使用局域网中任何一台 Linux 计算机运行 `ping getnas.local` 命令即可看到成功连通的输出。

由于 Windows 系统目前还不能识别 NAS 的主机名，因此进一步的测试我们等到配置 Samba 共享时再做介绍。

## 通过 SSH 远程登录 NAS 主机

## 数据盘配置