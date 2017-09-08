# 配置数据盘

数据存储是 NAS 服务器最基本也是最核心的功能。

我们将 Debian 系统安装在单独的 U 盘中，主要目的就是为了将操作系统与数据存储 `物理隔离`，力争做到当操作系统损坏时不伤及数据。

## 查看已安装磁盘

`fdisk` 命令搭配 `-l` 参数可查看主机所有磁盘和分区情况：

```
getnas@getnas:~$ sudo fdisk -l
Disk /dev/sda: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xb0b3946c


Disk /dev/sdb: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0xfb14eef7


Disk /dev/sdc: 7.3 GiB, 7807696896 bytes, 15249408 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf008b997

Device     Boot Start      End  Sectors  Size Id Type
/dev/sdc1  *     2048 13152255 13150208  6.3G 83 Linux
```

可以看到 NAS 服务器上目前有 3 个磁盘设备：

* 磁盘 `/dev/sda` 和 `/dev/sdb` 容量均 931.5 GiB，是我们用作存储数据的机械硬盘。
* 磁盘 `/dev/sda` 容量 7.3 GiB，是安装了 Debian 系统的 U 盘。

## 初始化磁盘

不论你所准备的硬盘新旧与否，我们都建议在创建存储分区之前对每块硬盘进行初始化并重新分区，让它们在相同的规格下工作。

### 将硬盘转换为 GPT 类型

考虑到有读者会使用 3TB 以上的硬盘，而 `MSDOS` 分区表单个分区最大仅支持 2TB。因此，为了 NAS 服务器未来扩容更操作更平滑，建议将所有硬盘的分区表都转换成 `GPT` 格式，它支持单个分区最大 18EB (1EB=1024PB=1,048,576TB)。

管理 `GPT` 格式的磁盘，需要使用 `Parted` 工具，安装：

```
getnas@getnas:~$ sudo apt install parted
```

下面以初始化 `/dev/sda` 硬盘为例，介绍如何使用用 `Parted`。

**第一步 在交互模式下打开 parted**

```
getnas@getnas:~$ sudo parted /dev/sda
GNU Parted 3.2
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

输入 `h` 或 `help` 查看可以执行的操作：

```
......
(parted) h
  align-check TYPE N                        check partition N for TYPE(min|opt) alignment
  help [COMMAND]                           print general help, or help on COMMAND
  mklabel,mktable LABEL-TYPE               create a new disklabel (partition table)
  mkpart PART-TYPE [FS-TYPE] START END     make a partition
  name NUMBER NAME                         name partition NUMBER as NAME
  print [devices|free|list,all|NUMBER]     display the partition table, available devices,
        free space, all found partitions, or a particular partition
  quit                                     exit program
......
(parted)
```

**第二步 磁盘转换成 GPT 格式**

输入 `mklabel gpt`：

```
(parted) mklabel gpt
Warning: The existing disk label on /dev/sda will be destroyed and all data on this disk will
be lost. Do you want to continue?
Yes/No? 
```

看到 `Yes/No` 提示时，输入 `yes` 并 `Enter` 回车键确认。

> 提示：如果需要将硬盘从 GPT 类型转换会 MBR 类型，只需执行 `mklabel msdos` 即可。

**第三步 检查是否转换成功**

输入 `p` 或 `print` 打印当前磁盘相关信息：

```
(parted) p
Model: ATA WDC WD10EZEX-00B (scsi)
Disk /dev/sda: 1000GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size   File system  Name     Flags
 1      1049kB  500GB   500GB  ext4         primary
 2      500GB   1000GB  500GB  ext4         primary

(parted)
```

看到 `Partition Table` 为 `gpt`，代表已经转换成功了。

**第四步 切换设备**

Parted 工具交互模式下每次操作一个磁盘设备，使用 `select` 命令可切换到其他磁盘设备，例如，切换成 `/dev/sdb`：

```
(parted) select
New device?  [/dev/sda]? /dev/sdb
Using /dev/sdb
```

### 删除旧分区

如果你给 NAS 服务器安装了使用过且未格式化的旧硬盘，硬盘中很可能存在旧的分区。在 `parted` 交互模式下输入 `p` 或 `print`，能够看到当前操作的磁盘是否存在旧的分区。

```
......
Disk Flags:

Number  Start   End     Size   File system  Name     Flags
 1      1049kB  500GB   500GB  ext4         primary
 2      500GB   1000GB  500GB  ext4         primary
```

例如当前硬盘中存在 2 个均为 500GB 的分区，数字编号分别为 `1` 和 `2`。使用 `rm` 命令加分区数字编号，即可删除分区。

```
(parted) rm 1
``` 

一次删除一个分区，可以反复用 `p` 或 `print` 命令查看磁盘信息，以确认分区是否成功删除。

## 存储方案

硬盘初始化完成，接下来需要我们思考并选择一个适合自己的存储方案。通常有以下几种方案供你选择：

* 方案一：HDD，数据直接存储在硬盘上
* 方案二：RAID，数据存储在由两块以上硬盘组成 RAID 磁盘阵列，数据更安全；
* 方案三：HDD + LVM，将硬盘添加到 LVM 存储池，可随时添加磁盘扩展存储容量；
* 方案四：RAID + LVM，将硬盘组成 RAID 磁盘阵列，再将磁盘阵列添加到 LVM 存储池，兼顾扩容能力和数据安全；

以上几种方案各有长短，没有所谓最佳方案，请读者根据对 NAS 服务器的实际需求进行选择。

例如，你计划备份和共享比较重要的照片，总量预计不会超过 4TB，那么用两块 4TB 硬盘组成 RAID 1 磁盘阵列就是不错的方案。

再比如，你计划用 NAS 做 BT 下载机，数据安全性要求不高，使用一块容量足够大的硬盘直接用作数据存储即可。

## 方案一 直接使用硬盘

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

## 方案二 硬盘 + LVM

配置 LVM 卷需要使用 `lvm2` 工具，安装：

```
getnas@getnas:~$ sudo apt install lvm2
```

## 方案三 直接使用 RAID 磁盘阵列

### 硬盘分区

**第二步 在交互模式下打开 gdisk**

**第三步 创建新分区**

### 创建 RAID 1

磁盘初始化完成后，接下来我们要把 NAS 服务器上的两块 1TB 硬盘配置成 RAID 1 磁盘阵列。

RAID 1 磁盘阵列是将两块硬盘互做镜像，任何数据都会以完全相同的方式分别存储到两块硬盘当中。即使其中一块硬盘损坏，也不会有数据丢失的风险。

> 注意：两块 1TB 的硬盘组成 RAID 1 磁盘阵列后，实际存储容量为单块硬盘的大小，即可用空间为 1TB。

我们需要使用 `mdadm` 工具配置磁盘阵列，安装：

```
getnas@getnas:~$ sudo apt install mdadm
```

使用 `mdadm` 命令附加必要的参数：

* `--create /dev/md0`：创建名为 `md0` 的磁盘阵列设备
* `--level=1`：阵列类型为 `raid 1`
* `--raid-devices=2`：指定该阵列由 2 个磁盘组成
* `/dev/sdX`：为组成阵列的磁盘路径名

```
getnas@getnas:~$ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
mdadm: /dev/sdb appears to be part of a raid array:
       level=raid0 devices=0 ctime=Thu Jan  1 08:00:00 1970
mdadm: partition table exists on /dev/sdb but will be lost or
       meaningless after creating array
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: /dev/sdc appears to be part of a raid array:
       level=raid0 devices=0 ctime=Thu Jan  1 08:00:00 1970
mdadm: partition table exists on /dev/sdc but will be lost or
       meaningless after creating array
Continue creating array?
```

看到 `Continue creating array?` 提示时输入 `yes` 并回车键确认继续。

```
......
Continue creating array? yes
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

这样，路径名为 `/dev/md0` 的 RAID 1 类型磁盘阵列就创建好了。

### 查看磁盘阵列信息

`mdadm` 命令后跟磁盘阵列设备名，即可查看最基本的设备信息：

```
getnas@getnas:~$ sudo mdadm /dev/md0
/dev/md0: 931.39GiB raid1 2 devices, 0 spares. Use mdadm --detail for more detail.
```

如果想查看更详尽的信息，可以在命令中添加 `--detail` 参数：

```
getnas@getnas:~$ sudo mdadm --detail /dev/md0
/dev/md0:
        Version : 1.2
  Creation Time : Thu Sep  7 17:57:33 2017
     Raid Level : raid1
     Array Size : 976631488 (931.39 GiB 1000.07 GB)
  Used Dev Size : 976631488 (931.39 GiB 1000.07 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

  Intent Bitmap : Internal

    Update Time : Thu Sep  7 17:59:54 2017
          State : clean, resyncing
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

  Resync Status : 2% complete

           Name : getnas:0  (local to host getnas)
           UUID : 054d4a70:e34ca554:9e07e4a7:a3ac28d9
         Events : 28

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
```

当我们再次使用 `fdisk -l` 命令时，即可看到名为 `/dev/md0` 的新设备：

```
getnas@getnas:~$ sudo fdisk -l
......
Disk /dev/md0: 931.4 GiB, 1000070643712 bytes, 1953262976 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

### 查看磁盘阵列状态

```

```

## 方案四 RAID + LVM