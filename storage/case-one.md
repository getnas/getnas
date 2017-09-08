# 方案一 直接使用硬盘

直接使用硬盘分区存储数据，可以在一块硬盘上创建多个分区，分区的最大容量等于硬盘的总可用容量。如果有多块硬盘，只能分别进行分区和挂载。

这种数据存储方案配置最简单，但没有冗余能力，硬盘一旦损坏，数据很难被恢复。适合存储非关键数据，比如软件、电影、音乐以及那些容易再次获得的数据。

我们以 `/dev/sda` 1TB 硬盘为例，介绍如何创建和挂载分区。

### 第一步 交互模式运行 Parted

```
getnas@getnas:~$ sudo parted /dev/sda
GNU Parted 3.2
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

### 第二步 创建分区

我们输入 `mkpart` 命令，创建一个占用整个硬盘空间的分区：

* Partition name：分区名，留空；
* File system type：文件系统类型，建议使用 ext4；
* Start：存储空间的起始位置，输入 `1`；
* End：存储空间的结束位置，输入 `-1` 即硬盘末尾；

```
(parted) mkpart
Partition name?  []?
File system type?  [ext2]? ext4
Start? 1
End? -1
```

输入 `p` 或 `print` 命令查看分区是否创建成功：

```
(parted) p
Model: ATA ST1000DM003-9YN1 (scsi)
Disk /dev/sda: 1000GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  1000GB  1000GB  ext4
```

可以看到分区已经成功创建，输入 `q` 或 `quit` 命令退出 parted。

### 第三步 格式化分区

使用 `fdisk` 命令查看 `/dev/sda` 设备的分区情况：

```
getnas@getnas:~$ sudo fdisk -l /dev/sda
Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 4336065F-D40A-4683-A837-B854017AC3CF

Device     Start        End    Sectors   Size Type
/dev/sda1   2048 1953523711 1953521664 931.5G Linux filesystem
```

可以看到，我们新创建的分区路径名为 `/dev/sda1`。

使用 `mkfs.ext4` 命令格式化分区：

```
getnas@getnas:~$ sudo mkfs.ext4 /dev/sda1
mke2fs 1.43.4 (31-Jan-2017)
/dev/sda1 有一个标签为“系统保留”的 ntfs 文件系统
Proceed anyway? (y,N) y
创建含有 244190208 个块（每块 4k）和 61054976 个inode的文件系统
文件系统UUID：12c59881-01cd-483e-969b-19e55e4e65ed
超级块的备份存储于下列块：
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848

正在分配组表： 完成
正在写入inode表： 完成
创建日志（262144 个块）完成
写入超级块和文件系统账户统计信息： 已完成
```

### 第四步 挂载分区

一般而言，在 Linux 系统中用户习惯挂载数据分区到 `/mnt` 目录下。

首先在此目录下创建一个名为 `storage` 的文件夹：

```
getnas@getnas:~$ sudo mkdir /mnt/storage
```

然后将 `/dev/sda1` 分区挂载到 `/mnt/storage` 目录：

```
getnas@getnas:~$ sudo mount /dev/sda1 /mnt/storage/
```

使用 `ls` 命令查看 `/mnt/storage` 目录，可以看到目录中出现了一个名为 `lost+found` 的文件夹：

```
getnas@getnas:~$ ls /mnt/storage/
lost+found
```

使用 `df` 命令加 `-h` 参数查看分区挂载情况：

```
getnas@getnas:~$ sudo df -h
文件系统         容量    已用  可用  已用% 挂载点
......
tmpfs           5.9G     0  5.9G    0% /sys/fs/cgroup
tmpfs           1.2G     0  1.2G    0% /run/user/1000
/dev/sda1       916G   77M  870G    1% /mnt/storage
```

从最后一行可以看到，`/dev/sda1` 分区的挂载点为 `/mnt/storage`。即所有存储到 `/mnt/storage` 文件夹中的数据，实际存储在 `/dev/sda1` 分区中。

### 第五步 设置分区的所有者权限

由于我们今后主要以 `getnas` 用户身份使用挂载的数据存储目录，因此很有必要重新指定目录的所有者：

```
getnas@getnas:~$ sudo chown getnas:getnas /mnt/storage
```

这样一来，我们无需在命令中添加 `sudo`，也能自由的在 `/mnt/storage` 目录中进行读、写、执行等操作。

### 第六步 分区信息写入 /etc/fstab 配置文件

手动挂载的分区在系统重启后不会自动重新挂载，为了实现分区的自动挂载，我们需要将挂载信息写入系统专门管理分区挂载信息的配置文件 `/etc/fstab` 当中。

在编辑配置文件之前，我们要使用 `blkid` 命令查看 `/dev/sda1` 分区的 `UUID` 信息：

```
getnas@getnas:~$ sudo blkid
/dev/sda1: UUID="12c59881-01cd-483e-969b-19e55e4e65ed" TYPE="ext4" PARTUUID="de13f291-3003-4cc6-a400-1a1c2dad2966"
```

使用 `nano` 编辑器打开配置文件：

```
getnas@getnas:~$ sudo nano /etc/fstab
```

在配置问的末尾新增一行，内容为：

```
UUID=12c59881-01cd-483e-969b-19e55e4e65ed  /mnt/storage  ext4  auto  0  0
```

> 注意：一行共有 6 个部分，用空格分隔每个部分，请用你实际查询到值替换 `UUID` 值。

配置好的文件看起来类似下面这样，带有 `#` 号的行为注释信息：

```
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
UUID=a915e0e5-6249-42ec-8be0-2624f3511275 /               ext4    errors=remount-ro 0       1
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
UUID=12c59881-01cd-483e-969b-19e55e4e65ed  /mnt/storage  ext4  auto  0  0
```

编辑好以后使用组合键 `CTRL + o` 保存，使用组合键 `CTRL + x` 退出编辑器。
