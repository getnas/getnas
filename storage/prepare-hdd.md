# 准备数据盘

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

## 选择存储方案

硬盘初始化完成，接下来需要我们思考并选择一个适合自己的存储方案。通常有以下几种方案供你选择：

* 方案一：HDD，数据直接存储在硬盘上
* 方案二：RAID，数据存储在由两块以上硬盘组成 RAID 磁盘阵列，数据更安全；
* 方案三：HDD + LVM，将硬盘添加到 LVM 存储池，可随时添加磁盘扩展存储容量；
* 方案四：RAID + LVM，将硬盘组成 RAID 磁盘阵列，再将磁盘阵列添加到 LVM 存储池，兼顾扩容能力和数据安全；

以上几种方案各有长短，没有所谓最佳方案，请读者根据对 NAS 服务器的实际需求进行选择。

例如，你计划备份和共享比较重要的照片，总量预计不会超过 4TB，那么用两块 4TB 硬盘组成 RAID 1 磁盘阵列就是不错的方案。

再比如，你计划用 NAS 做 BT 下载机，数据安全性要求不高，使用一块容量足够大的硬盘直接用作数据存储即可。