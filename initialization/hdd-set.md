# 配置数据盘

数据存储是 NAS 服务器最基本也是最核心的功能。

我们将 Debian 系统安装在单独的 U 盘中，主要目的就是为了将操作系统与数据存储 `物理隔离`，力争做到当操作系统损坏时不伤及数据。

## 查看已安装磁盘

`fdisk` 命令搭配 `-l` 参数可查看主机所有磁盘和分区情况：

```
getnas@getnas:~$ sudo fdisk -l

Disk /dev/sda: 7.3 GiB, 7807696896 bytes, 15249408 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf008b997

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1  *     2048 13152255 13150208  6.3G 83 Linux


Disk /dev/sdb: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x9c4da728


Disk /dev/sdc: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x2fe06793
```

从命令输出可以看到 NAS 服务器上目前有 3 个磁盘设备：

* 磁盘 `/dev/sda` 容量 7.3 GiB，是安装了 Debian 系统的 U 盘。
* 磁盘 `/dev/sdb` 和 `/dev/sdc` 容量均 931.5 GiB，是我们准备用作存储数据的机械硬盘。

## 初始化磁盘

不论你所准备的硬盘新旧与否，我们都建议在创建存储分区之前对每块硬盘进行初始化并重新分区，让它们在相同的规格下工作。

### 将硬盘转换为 GPT 类型

考虑到会有读者使用 3TB 以上的硬盘用作数据存储，然而普通的 `MSDOS` 分区表单个分区最大仅支持 2TB。因此，这里我们将把所有硬盘的分区表都转换成 `GPT` 格式，它支持单个分区最大 18EB (1EB=1024PB=1,048,576TB)。

管理 `GPT` 格式的磁盘，需要使用 `Parted` 工具，安装：

```
getnas@getnas:~$ sudo apt install parted
```

这里我们以初始化 `/dev/sdb` 硬盘为例介绍如何使用用 `Parted`。

**第一步 交互模式管理磁盘**

```
getnas@getnas:~$ sudo parted /dev/sdb
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
Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will
be lost. Do you want to continue?
Yes/No? 
```

看到 `Yes/No` 提示时，输入 `yes` 并 `Enter` 回车键确认。

**第三步 检查是否转换成功**

输入 `p` 或 `print` 打印当前磁盘相关信息：

```
(parted) p
Model: ATA WDC WD10EZEX-00B (scsi)
Disk /dev/sdb: 1000GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size   File system  Name     Flags
 1      1049kB  500GB   500GB  ext4         primary
 2      500GB   1000GB  500GB  ext4         primary

(parted)
```

看到 `Partition Table` 为 `gpt`，代表已经转换成功了。

### 删除旧分区

如果你给 NAS 服务器安装了使用过且未格式化的旧硬盘，硬盘中很可能存在旧的分区。在 `parted` 交互模式下输入 `p` 或 `print`，能够看到当前操作的磁盘是否存在旧的分区。

从前面转换磁盘为 GPT 格式最后的输出可以看到，当前硬盘中存在 2 个均为 500GB 的分区，他们的数字编号分别为 `1` 和 `2`。使用 `rm` 命令加分区数字编号，即可快速删除分区，一次删除一个分区。

```
(parted) rm 1
``` 

## RAID - 磁盘阵列

### 硬盘分区

使用 `gdisk` 工具，在两块硬盘上分别创建 `Linux RAID` 类型的分区。

**第一步 安装 gdisk**

```
getnas@getnas:~$ sudo apt install gdisk
```

**第二步 在交互模式下打开 gdisk**

```
getnas@getnas:~$ sudo gdisk /dev/sdb
[sudo] getnas 的密码：
GPT fdisk (gdisk) version 1.0.1

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):
```

输入 `?` 或 `h` 或 `help` 查看可用的命令：

```
Command (? for help): h
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu
```

**第三步 创建新分区**

输入 `n` 回车确认，开始创建新分区：

* Partition number：分区编号，默认，直接按回车确认；
* First sector：分区在磁盘的起始位置，默认，直接回车确认；
* Last sector：分区在磁盘的结束位置，保留硬盘最后的 100MB 空间，输入 `-100M`；
* Hex code or GUID：分区类型的 16 进制代码，输入 `fd00`，即 `Linux RAID` 类型；

```
Command (? for help): n
Partition number (1-128, default 1):
First sector (34-1953525134, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-1953525134, default = 1953525134) or {+-}size{KMGTP}: -100MB
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): fd00
Changed type of partition to 'Linux RAID'
```

**第四步 将所有变化写入磁盘**

前面所做的变更临时存储在内存中，如果想让新分区生效，需要将所有变更写入磁盘。

输入 `w`，程序会给出简短的安全提示，并询问我们是否继续执行，输入 `y` 并按回车确认。

```
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
```

写入完成后，程序自动退出。

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

### 将磁盘阵列信息写入配置文件

新建的磁盘阵列设备 `/dev/md0` 信息必须写入到 `mdadm` 管理工具的配置文件中才能确保系统重启后磁盘阵列设备能够被正常识别和使用。

切换到 `root` 用户身份：

```
getnas@getnas:~$ sudo su
```

将 `/dev/md0` 设备信息写入 `mdadm.conf` 配置文件：

```
root@getnas:/home/getnas# mdadm --detail --scan /dev/md0 >> /etc/mdadm/mdadm.conf
```

退回普通用户身份，使用组合键 `CTRL + D` 或输入 `exit` 回车键确认。

有兴趣的读者可以尝试不写阵列信息到配置文件，创建磁盘阵列后直接重新启动。你会发现重启系统后磁盘阵列设备已经不存在了。

### 查看磁盘阵列状态

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
Disk /dev/sda: 7.3 GiB, 7807696896 bytes, 15249408 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf008b997

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1  *     2048 13152255 13150208  6.3G 83 Linux


The primary GPT table is corrupt, but the backup appears OK, so that will be used.
Disk /dev/sdb: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: B8DF590E-740A-4FEF-9FB1-8F84C508BB84


The primary GPT table is corrupt, but the backup appears OK, so that will be used.
Disk /dev/sdc: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: C371F528-85BC-49EB-A28E-753EB84E105F


Disk /dev/md0: 931.4 GiB, 1000070643712 bytes, 1953262976 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
```

## 配置 LVM

配置 LVM 卷需要使用 `lvm2` 工具，安装：

```
getnas@getnas:~$ sudo apt install lvm2
```