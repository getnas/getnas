# 方案三 直接使用 RAID 磁盘阵列

### 概述

RAID - 磁盘阵列，是一种将多个硬盘组成具有冗余能力的阵列设备进行使用的技术。磁盘阵列有软硬之分，通过操作系统中的软件实现的称为软阵列(Soft-RAID)，由外置卡管理的称为硬阵列(RAID)。通常有以下几种模式：

* RAID 0：由 2 块以上的硬盘组成，容量为所有硬盘的容量之和，没有冗余和错误修复能力，磁盘性能和吞吐量最佳，阵列中任何一块硬盘发生损坏，整个阵列设备中的数据都将丢失。这种形式的磁盘阵列只适合存储那些安全要求最低的数据。
* RAID 1：由 2 块以上硬盘组成，最高可用容量为磁盘总容量的 50%，每块硬盘之间互做镜像，任何数据都会在阵列中的每一块硬盘上存取。磁盘利用率较低，性能和吞吐量较差，但冗余能力最佳。适合存储安全要求较高的重要数据。
* RAID 5：由 3 块以上硬盘组成，每块硬盘都有一部分空间被用户数据校验，实际会有一块硬盘容量的空间被用作数据校验，剩余的容量为可用存储，任何一块硬盘损坏不会丢失数据。这种模式兼顾了冗余和性能，适合存储安全性要求较高的数据。
* RAID 10：是由 2 个 RAID 1 组成的 RAID 0。这种形式即能获得 RAID 1 的冗余能力，又能获得 RAID 0 磁盘性能，可用容量为磁盘总容量的 50%。

## 准备工作

配置磁盘阵列的过程中我们需要用到 `parted` 分区工具，和 `mdadm` 磁盘阵列管理工具，使用以下命令安装：

```
getnas@getnas:~$ sudo apt install parted mdadm
```

## 硬盘分区

为了让系统能够妥善的识别我们创建的磁盘阵列，在开始之前应该先对磁盘进行分区。可以使用 `parted` 也可以使用 `gdisk`。

### 第一步 在交互模式下打开 parted

以 `/dev/sda` 为例：

```
getnas@getnas:~$ sudo parted /dev/sda
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

### 第二步 创建新分区

首先使用 `p` 命令查看该设备下是否存在旧分区，如果存在，使用 `rm` 命令将其删除。

```
(parted) p
Model: ATA ST1000DM003-9YN1 (scsi)
Disk /dev/sda: 1000GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags

(parted)
```

确认设备中没有旧分区，使用 `mkpart` 命令创建新分区，在交互过程中使用采用以下设置，创建一个占用整个磁盘容量的新分区：

* Partition name：留空；
* File system type：输入 `ext4`；
* Start：分区的起始位置，输入 `2048`；
* End：分区的结束位置，输入 `-1` 代表硬盘的最末端；

```
(parted) mkpart
Partition name?  []?
File system type?  [ext2]? ext4
Start? 2048
End? -1

(parted)
```

分区完成后可以再次使用 `p` 命令，查看是否创建成功。以此类推，重复上述步骤为 `/dev/sdb` 等硬盘创建新分区。输入 `q` 退出程序。

## 创建 RAID 1

接下来我们将 `/dev/sda1` 和 `/dev/sdb1` 这两个均为 1TB 的分区组成 RAID 1 磁盘阵列。

创建磁盘阵列使用 `mdadm` 命令附加必要的参数：

* `--create /dev/md0`：创建名为 `md0` 的磁盘阵列设备；
* `--level=1`：阵列类型为 `raid 1`；
* `--raid-devices=2`：指定该磁盘阵列由 2 个磁盘设备组成；
* `/dev/sdX`：组成阵列的磁盘分区；

```
getnas@getnas:~$ sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```
看到 `Continue creating array?` 交互信息时输入 `y` 或 `yes` 确认。

这样就完成了 RAID 1 磁盘阵列的创建。

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
  Creation Time : Wed Sep 13 22:02:33 2017
     Raid Level : raid1
     Array Size : 974630912 (929.48 GiB 998.02 GB)
  Used Dev Size : 974630912 (929.48 GiB 998.02 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

  Intent Bitmap : Internal

    Update Time : Wed Sep 13 22:02:43 2017
          State : clean, resyncing
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

  Resync Status : 0% complete

           Name : getnas:0  (local to host getnas)
           UUID : 7ab3a467:4d1f7403:a38385b4:eca86e95
         Events : 2

    Number   Major   Minor   RaidDevice State
       0       8        1        0      active sync   /dev/sda1
       1       8       17        1      active sync   /dev/sdb1
```

从输出的信息中可以看到，磁盘阵列的名称(Name)为 `getnas:0`，你的磁盘阵列名称可能与此处的不同，请使用实际的名称。

### 格式化并挂载分区

目前设备的路径名仍然为 `/dev/md0`，但当我们重新启动系统以后，设备名就会变成 `/dev/md/getnas:0`。此处，我们以重启之前进行格式化和挂载举例。

**第一步 格式化分区**

使用 `mkfs.ext4` 命令格式化分区：

```
getnas@getnas:~$ sudo mkfs.ext4 /dev/md0
创建含有 243657728 个块（每块 4k）和 60915712 个inode的文件系统
文件系统UUID：a80571bc-345c-4a62-9dd3-22f98c63f679
超级块的备份存储于下列块：
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848

正在分配组表： 完成
正在写入inode表： 完成
创建日志（262144 个块）完成
写入超级块和文件系统账户统计信息： 已完成
```

**第二步 创建挂载目录**

在挂载分区之前，先确认是否创建了 `/mnt/storage` 目录，如果没有请执行以下命令创建：

```
getnas@getnas:~$ sudo mkdir /mnt/storage
```

**第三步 手动挂载分区**

```
getnas@getnas:~$ sudo mount /dev/md0 /mnt/storage
```

**第四步 设置目录所有权**

使用 `chown` 命令，将 `/mnt/storage` 目录的所有者指定为 `getnas`。请使用你在安装系统时实际创建的账户名替换该用户名：

```
getnas@getnas:~$ sudo chown getnas /mnt/storage
```

### 配置分区自动挂载

使用 `nano` 编辑器打开配置文件 `/etc/fstab`：

```
getnas@getnas:~$ sudo nano /etc/fstab
```

在配置文件的最后添加一行：

```
/dev/md/getnas:0  /mnt/storage  ext4  auto  0  0
```

> 注意：请将分区名称中的 `getnas:0` 替换成你真实的磁盘阵列名。

编辑完成以后，配置文件开起来应该类似这样，其中 `#` 开头的行为注释：

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
/dev/md/getnas:0  /mnt/storage  ext4  auto  0  0
```

重新启动系统，使用 `df -h` 命令即可查看分区挂载情况。
