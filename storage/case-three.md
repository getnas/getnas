# 方案三 直接使用 RAID 磁盘阵列

### 概述

RAID - 磁盘阵列，是一种将多个硬盘组成具有冗余能力的阵列设备进行使用的技术。磁盘阵列有软硬之分，通过操作系统中的软件实现的称为软阵列(Soft-RAID)，由外置卡管理的称为硬阵列(RAID)。通常有以下几种模式：

* RAID 0：由 2 块以上的硬盘组成，容量为所有硬盘的容量之和，没有冗余和错误修复能力，磁盘性能和吞吐量最佳，阵列中任何一块硬盘发生损坏，整个阵列设备中的数据都将丢失。这种形式的磁盘阵列只适合存储那些安全要求最低的数据。
* RAID 1：由 2 块以上硬盘组成，最高可用容量为磁盘总容量的 50%，每块硬盘之间互做镜像，任何数据都会在阵列中的每一块硬盘上存取。磁盘利用率较低，性能和吞吐量较差，但冗余能力最佳。适合存储安全要求较高的重要数据。
* RAID 5：由 3 块以上硬盘组成，每块硬盘都有一部分空间被用户数据校验，实际会有一块硬盘容量的空间被用作数据校验，剩余的容量为可用存储，任何一块硬盘损坏不会丢失数据。这种模式兼顾了冗余和性能，适合存储安全性要求较高的数据。
* RAID 10：是由 2 个 RAID 1 组成的 RAID 0。这种形式即能获得 RAID 1 的冗余能力，又能获得 RAID 0 磁盘性能，可用容量为磁盘总容量的 50%。

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

分区完成后可以再次使用 `p` 命令，查看是否创建成功。以此类推，重复上述步骤为 `/dev/sdb` 等硬盘创建新分区。

## 创建 RAID 1

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
