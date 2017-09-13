# 方案二 硬盘 + LVM

第一种存储方案是直接在硬盘上创建分区，单分区的最大容量不能超过硬盘容量。当分区容量被用尽，需要用容量更大的硬盘进行替换，这种替换操作不但耗时，而且存在很大的安全风险。

Logical Volume Manager (逻辑卷管理器) - LVM，是专门用来解决分区扩容问题的机制。LVM 是介于硬盘和分区之间的一个逻辑层，举个例子，如果把硬盘比作电池，那么 LVM 就是由多节电池并联组成的电池组，增加电池即可增大容量。

## 安装 LVM 管理工具

配置 LVM 卷需要使用 `lvm2` 工具，安装：

```
getnas@getnas:~$ sudo apt install lvm2
```

## 几个基本概念

LVM 中有三个最基本的概念，使用之前必须了解：

* PV : Physical Volumes - 物理卷。包括硬盘、分区、RAID 以及 SAN 提供的 LUN 等。
* VG : Volume Groups - 卷组。由一个或多个物理卷组成。
* LV : Logical Volumes - 逻辑卷。在卷组上创建的虚拟分区。

## 创建 Linux LVM 类型的分区

虽然将未分区的硬盘创建成 PV 符合技术定义，但这样操作存在一定的安全隐患。使用分区工具时可能会误以为那是一块未被使用的新硬盘，这就很容易导致误操作而丢失数据。建议读者先给硬盘创建分区，再用分区创建 PV。

创建 `Linux LVM` 类型的分区，我们可以使用 `gdisk` 分区工具

### 第一步 安装 gdisk

```
getnas@getnas:~$ sudo apt install gdisk
```

### 第二步 交互方式打开 gdisk

> 在使用 gdisk 分区之前，请先参考“[准备数据盘](prepare-hdd.md)”初始化硬盘，删除硬盘中的旧分区。

我们以 `/dev/sda` 为例：

```
getnas@getnas:~$ sudo gdisk /dev/sda
GPT fdisk (gdisk) version 1.0.1

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help):
```

输入 `p`，查看并确认磁盘中没有旧的分区：

```
Command (? for help): p
Disk /dev/sda: 1953525168 sectors, 931.5 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 4336065F-D40A-4683-A837-B854017AC3CF
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 1953525134
Partitions will be aligned on 2048-sector boundaries
Total free space is 1953525101 sectors (931.5 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help):
```

### 第三步 创建新分区

输入 `n`，创建新分区，下面使用整个硬盘空间创建一个 `Linux LVM` 类型的分区：

* Partition number：分区编号，按回车使用默认值；
* First sector：分区起始扇区，按回车使用默认值；
* Last sector：分区结束扇区，按回车使用默认值；
* Hex code or GUID：分区代码，输入 `8e00`;

```
Command (? for help): n
Partition number (1-128, default 1):
First sector (34-1953525134, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-1953525134, default = 1953525134) or {+-}size{KMGTP}:
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8e00
Changed type of partition to 'Linux LVM'

Command (? for help):
```

再次输入 `p`，查看分区是否创建成功：

```
Command (? for help): p
Disk /dev/sda: 1953525168 sectors, 931.5 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 4336065F-D40A-4683-A837-B854017AC3CF
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 1953525134
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048      1953525134   931.5 GiB   8E00  Linux LVM

Command (? for help):
```

### 第四步 写入分区并退出

要前面的分区操作生效，需要使用 `w` 指令将信息写入硬盘：

```
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!
```

程序会进一步询问是否确认执行，输入 `y`：

```
Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sda.
Warning: The kernel is still using the old partition table.
The new table will be used at the next reboot or after you
run partprobe(8) or kpartx(8)
The operation has completed successfully.
```

操作完成后程序会自动退出。其他硬盘的分区方法完全相同，重复以上四个步骤即可。

## 使用 LVM

### 第一步 扫描可以用作 LVM 的磁盘

`LVM2` 包中提供了一系列管理 LVM 的工具，其中 `lvmdiskscan` 命令用作扫描可用作 LVM 的磁盘：

```
getnas@getnas:~$ sudo lvmdiskscan
  /dev/sda1 [     931.51 GiB]
  /dev/sdb1 [     931.51 GiB]
  /dev/sdc1 [       6.27 GiB]
  0 disks
  3 partitions
  0 LVM physical volume whole disks
  0 LVM physical volumes
```

扫描结果显示，当前 NAS 服务器上可以用作 LVM 的有 3 个分区，其中 `/dev/sda1` 和 `/dev/sdb1` 是我们创建的 `Linux LVM` 类型的新分区。`/dev/sdc1` 是系统所在 U 盘上的分区。

### 第二步 将分区创建为 PV

以 `/dev/sda1` 为例，使用 `pvcreate` 命令创建 PV：

```
getnas@getnas:~$ sudo pvcreate /dev/sda1
WARNING: ext4 signature detected on /dev/sda1 at offset 1080. Wipe it? [y/n]: y
  Wiping ext4 signature on /dev/sda1.
  Physical volume "/dev/sda1" successfully created.
```

创建过程会发出警告，该操作会抹去原有的磁盘的类型信息，输入 `y` 确认。

使用 `pvscan` 或 `pvs` 命令可以查看已经创建的 PV：

```
getnas@getnas:~$ sudo pvscan
  PV /dev/sda1                      lvm2 [931.51 GiB]
  Total: 1 [931.51 GiB] / in use: 0 [0   ] / in no VG: 1 [931.51 GiB]

getnas@getnas:~$ sudo pvs
  PV         VG Fmt  Attr PSize   PFree
  /dev/sda1     lvm2 ---  931.51g 931.51g
```

### 第三步 将 PV 创建为 VG

下面假设要使用 `/dev/sda1` PV 创建一个名为 `vg-1` 的 VG，使用 `vgcreate` 命令：

```
getnas@getnas:~$ sudo vgcreate vg-1 /dev/sda1
  Volume group "vg-1" successfully created
```

使用 `vgscan` 或 `vgs` 命令可以查看已经创建的 VG：

```
getnas@getnas:~$ sudo vgscan
  Reading volume groups from cache.
  Found volume group "vg-1" using metadata type lvm2

getnas@getnas:~$ sudo vgs
  VG   #PV #LV #SN Attr   VSize   VFree
  vg-1   1   0   0 wz--n- 931.51g 931.51g
```

### 第四步 在 VG 上创建 LV

使用 `lvcreate` 命令：

* `-n` 或 `--name`：LV 逻辑卷名称；
* `-l` 或 `--extents`：LV 的容量，只能小于或等于 VG 容量。使用 `100%FREE` 代表使用 VG 上所有可用空间；
* `-L` 或 `--size`：指定 LV 的确切容量，如：-L 100g；

以下命令在 `vg-1` 上创建了一个名为 `lv-storage` 且使用全部 VG 容量的 LV：

```
getnas@getnas:~$ sudo lvcreate -n lv-storage -l 100%FREE vg-1
  Logical volume "lv-storage" created.
```

使用 `lvs` 或 `lvscan` 查看已经创建的 LV：

```
getnas@getnas:~$ sudo lvs
  LV         VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-storage vg-1 -wi-a----- 931.51g
  
getnas@getnas:~$ sudo lvscan
  ACTIVE            '/dev/vg-1/lv-storage' [931.51 GiB] inherit
```

现在，就可以像普通分区一样去使用新创建的逻辑卷 `/dev/vg-1/lv-storage`。

### 第五步 格式化并挂载 LV 

使用 `mkfs.ext4` 命令将 `/dev/vg-1/lv-storage` 格式化为 `ext4` 类型：

```
getnas@getnas:~$ sudo mkfs.ext4 /dev/vg-1/lv-storage
mke2fs 1.43.4 (31-Jan-2017)
创建含有 244189184 个块（每块 4k）和 61054976 个inode的文件系统
文件系统UUID：75c7f79c-f798-4d1e-bd07-45be15eaad97
超级块的备份存储于下列块：
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848

正在分配组表： 完成
正在写入inode表： 完成
创建日志（262144 个块）完成
写入超级块和文件系统账户统计信息： 已完成
```

接下来，我们要挂载 LV 分区，如果尚未创建 `/mnt/storage` 目录，请使用 `mkdir` 命令创建：

```
getnas@getnas:~$ sudo mkdir /mnt/storage
```

将 LV 手动挂载到 `/mnt/storage` 目录：

```
getnas@getnas:~$ sudo mount /dev/vg-1/lv-storage /mnt/storage/
```

使用 `df -h` 命令可以查看分区挂载情况：

```
getnas@getnas:~$ df -h
文件系统                       容量  已用  可用 已用% 挂载点
udev                           5.9G     0  5.9G    0% /dev
......
/dev/mapper/vg--1-lv--storage  916G   77M  870G    1% /mnt/storage
```

将分区所有者设置为 `getnas`，以便于我们以 `getnas` 用户身份管理该目录时有全部的权限：

```
getnas@getnas:~$ sudo chown getnas /mnt/storage/
```

### 第六步 分区信息写入 /etc/fstab 配置文件

手动挂载的分区在系统重启后不会自动重新挂载，为了实现分区的自动挂载，我们需要将分区信息写入配置文件 /etc/fstab 当中。

使用 nano 编辑器打开配置文件：

```
getnas@getnas:~$ sudo nano /etc/fstab
```

在配置文件的最后添加一行：

```
/dev/vg-1/lv-storage  /mnt/storage  ext4  auto  0  0
```

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
/dev/vg-1/lv-storage  /mnt/storage  ext4  auto  0  0 
```

编辑好以后使用组合键 `CTRL + o` 保存，使用组合键 `CTRL + x` 退出编辑器。

## LVM 管理

### VG 扩容

使用 `vgdisplay` 命令查看 VG 的详细信息：

```
getnas@getnas:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               vg-1
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               931.51 GiB
  PE Size               4.00 MiB
  Total PE              238466
  Alloc PE / Size       238466 / 931.51 GiB
  Free  PE / Size       0 / 0
  VG UUID               hh1BR6-dpJH-6pXP-OMuK-Tt3j-IeL6-M8ryC7
```

从输出结果可以看到，`vg-1` 卷组的容量(VG Size)为 `931.51 GiB`，已分配(Alloc Size)为 `931.51 GiB`，剩余可用空间(Free Size)为 `0`。

可以向 VG 中添加 PV 增加可用空间。

根据之前介绍的方法，用 `pvcreate` 命令将 `/dev/sdb1` 分区也创建成 PV。

```
getnas@getnas:~$ sudo pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
```

使用 `pvdisplay` 命令可查看所有 PV 的详细信息：

```
getnas@getnas:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda1
  VG Name               vg-1
  PV Size               931.51 GiB / not usable 4.69 MiB
  Allocatable           yes (but full)
  PE Size               4.00 MiB
  Total PE              238466
  Free PE               0
  Allocated PE          238466
  PV UUID               UVixOQ-RH6R-X5Wd-xUt0-fRf0-03gv-aKjnzQ

  "/dev/sdb1" is a new physical volume of "931.51 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               931.51 GiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               WFFokx-wt5j-OJOv-nl0X-OTFF-nc3p-mS1GQL
```

从输出的信息可以看到，`/dev/sdb1` 是一个新创建的物理卷，使用 `vgextend` 命令将它添加到 `vg-1` 卷组中：

```
getnas@getnas:~$ sudo vgextend vg-1 /dev/sdb1
  Volume group "vg-1" successfully extended
```

再次使用 `vgdisplay` 命令，可以看到卷组 `vg-1` 当前(Cur PV)有 2 个，卷组总容量(VG Size)为 `1.82 TiB`，可用空间(Free Size) 为 `931.51 GiB`：

```
getnas@getnas:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               vg-1
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               1.82 TiB
  PE Size               4.00 MiB
  Total PE              476932
  Alloc PE / Size       238466 / 931.51 GiB
  Free  PE / Size       238466 / 931.51 GiB
  VG UUID               hh1BR6-dpJH-6pXP-OMuK-Tt3j-IeL6-M8ryC7
```

### LV 扩容

卷组扩容完成后，就可以给 LV 逻辑卷进行扩容了，首先使用 `lvdisplay` 命令查看逻辑卷的详细信息：

```
getnas@getnas:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg-1/lv-storage
  LV Name                lv-storage
  VG Name                vg-1
  LV UUID                EBD8c9-Phfx-VxT4-5bez-KPNg-yfdv-2NfHtQ
  LV Write Access        read/write
  LV Creation host, time getnas, 2017-09-13 12:12:34 +0800
  LV Status              available
  # open                 1
  LV Size                931.51 GiB
  Current LE             238466
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:0
```

可以看到，当前逻辑卷 `/dev/vg-1/lv-storage` 的容量为 `931.51 GiB`。

使用 `lvresize` 命令为其扩容，将 `vg-1` 卷组的所有可用空间分配给逻辑卷：

```
getnas@getnas:~$ sudo lvresize -l +100%FREE -r vg-1/lv-storage
  Size of logical volume vg-1/lv-storage changed from 931.51 GiB (238466 extents) to 1.82 TiB (476932 extents).
  Logical volume vg-1/lv-storage successfully resized.
```

> 注意：命令中的 `-l` 参数可以用 `--extends` 替代，`+100%FREE` 代表使用所有可用空间进行扩容，不要漏掉 `+` 号。`-r` 参数可以用 `--resizefs` 替代，用于调整文件系统大小以适合 LV 扩容后的容量。

再次使用 `lvdisplay` 命令查看逻辑卷信息，可以看到逻辑卷容量(LV Size) 已提升至 `1.82 TiB`：

```
getnas@getnas:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/vg-1/lv-storage
  LV Name                lv-storage
  VG Name                vg-1
  LV UUID                EBD8c9-Phfx-VxT4-5bez-KPNg-yfdv-2NfHtQ
  LV Write Access        read/write
  LV Creation host, time getnas, 2017-09-13 12:12:34 +0800
  LV Status              available
  # open                 1
  LV Size                1.82 TiB
  Current LE             476932
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           254:0
```

### 从 LVM 移除硬盘

有可能是硬盘故障，也可能是你想从卷组中拆一块硬盘出来挪作它用，不论出于什么原因，要从 LVM 中移除硬盘，必须确保满足以下条件：

* 即将从 LVM 中移除的硬盘空间最好未被使用；
* 如果硬盘空间已被使用，必须确保卷组中其他硬盘上的剩余空间足够存放将被拆除硬盘中已存储的数据；

接着前面的例子，我们已经使用两块 1TB 硬盘创建的 PV 组成了 `vg-1` 卷组，并且使用卷组的全部空间创建了逻辑卷 `lv-storage`。因为空间几乎没有使用，现在要把 `/dev/sdb1` 从 LVM 中移除。

**第一步 卸载分区**

```
getnas@getnas:~$ sudo umount /dev/vg-1/lv-storage
```

**第二步 缩减 LV 空间**

由于目前两块硬盘组成卷组的所有空间都被 `lv-storage` 逻辑卷占用了，因此必须先将其缩减。由于两块硬盘的容量相同，且逻辑卷占用改了两块硬盘的 100% 空间，因此我们要缩减 LV 总容量的 50%。使用 `lvresize` 命令：

```
getnas@getnas:~$ sudo lvresize -l -50%LV -r vg-1/lv-storage
fsck from util-linux 2.29.2
/dev/mapper/vg--1-lv--storage：13/122101760 文件（0.0% 为非连续的）， 10505021/488378368 块
resize2fs 1.43.4 (31-Jan-2017)
将 /dev/mapper/vg--1-lv--storage 上的文件系统调整为 244189184 个块（每块 4k）。
/dev/mapper/vg--1-lv--storage 上的文件系统现在为 244189184 个块（每块 4k）。

  Size of logical volume vg-1/lv-storage changed from 1.82 TiB (476932 extents) to 931.51 GiB (238466 extents).
  Logical volume vg-1/lv-storage successfully resized.
```

**第三步 转移数据**

使用 `pvmove` 命令将 `/dev/sdb1` 上的数据转移到 VG 中其他设备上：

```
getnas@getnas:~$ sudo pvmove /dev/sdb1
  No extents available for allocation
```

虽然被分配给了 LV ，但 `/dev/sdb1` 这个 PV 的空间尚未被占用，因此不需要转移数据。

> 注意：如果你的 PV 在逻辑卷中被存入了数据，那么在执行数据转移的时候，转移操作可能需要很长时间才能完成。

**第四步 从 VG 中移除 PV**

使用 `vgreduce` 命令，从 `vg-1` 中移除 `/dev/sdb1`：

```
getnas@getnas:~$ sudo vgreduce vg-1 /dev/sdb1
  Removed "/dev/sdb1" from volume group "vg-1"
```

**第五步 删除 PV**

使用 `pvremove` 命令，将 `/dev/sdb1` 上的 LVM 相关信息抹去：

```
getnas@getnas:~$ sudo pvremove /dev/sdb1
  Labels on physical volume "/dev/sdb1" successfully wiped.
```

好了，现在硬盘已经从 LVM 中成功移除。
