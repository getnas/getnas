# 方案四 RAID + LVM

这可能是几种方案中最复杂的一种，但有了 [方案二](case-two.md) 和 [方案三](case-three.md) 的基础，RAID + LVM 的配置可能也不会显得那么复杂了。

在方案三中我们已经配置了两块 1TB 硬盘组成的 RAID 1 磁盘阵列，我们只需要将它添加到 LVM 中即可实现两种技术的结合。

## 准备工作

本方案中的操作需要用到 `mdadm` 和 `gdisk` 程序，如果你的 NAS 服务器尚未安装请执行：

```
getnas@getnas:~$ sudo apt install mdadm gdisk
```

## 检查 RAID 状态

使用 `mdadm` 命令搭配 `--detail` 和 `--scan` 参数扫描 NAS 服务器中正在工作的磁盘阵列：

```
getnas@getnas:~$ sudo mdadm --detail --scan
ARRAY /dev/md/getnas:0 metadata=1.2 name=getnas:0 UUID=7ab3a467:4d1f7403:a38385b4:eca86e95
```

可以看到我们在方案三中创建的名为 `/dev/md/getnas:0` 的磁盘阵列，执行命令进一步查看磁盘阵列的详细信息：

```
getnas@getnas:~$ sudo mdadm --detail /dev/md/getnas:0
/dev/md/getnas:0:
        Version : 1.2
  Creation Time : Wed Sep 13 22:02:33 2017
     Raid Level : raid1
     Array Size : 974630912 (929.48 GiB 998.02 GB)
  Used Dev Size : 974630912 (929.48 GiB 998.02 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

  Intent Bitmap : Internal

    Update Time : Thu Sep 14 10:44:14 2017
          State : active, resyncing
 Active Devices : 2
Working Devices : 2
 Failed Devices : 0
  Spare Devices : 0

  Resync Status : 92% complete

           Name : getnas:0  (local to host getnas)
           UUID : 7ab3a467:4d1f7403:a38385b4:eca86e95
         Events : 1411

    Number   Major   Minor   RaidDevice State
       0       8        1        0      active sync   /dev/sda1
       1       8       17        1      active sync   /dev/sdb1
```

## 磁盘阵列分区

卸载已挂载的磁盘阵列设备：

```
getnas@getnas:~$ sudo umount /dev/md/getnas:0
```

使用 `gdisk` 交互模式下打开磁盘阵列设备：

```
getnas@getnas:~$ sudo gdisk /dev/md/getnas:0
GPT fdisk (gdisk) version 1.0.1

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.

Command (? for help): 
```

输入 `p` 查看设备分区情况：

```
Command (? for help): p
Disk /dev/md/getnas:0: 1949261824 sectors, 929.5 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): C06AA593-EF3A-4BA7-925A-17A691195FF2
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 1949261790
Partitions will be aligned on 2048-sector boundaries
Total free space is 1949261757 sectors (929.5 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name

Command (? for help): 
```

检查并确认该设备上不存在旧的分区，如果有，请使用 `d` 指令删除。

输入 `n` 创建新分区：

* Partition number - 分区编号，按回车使用默认值；
* First sector - 起始扇区，按回车使用默认值；
* Last sector - 结束删除，按回车使用默认值；
* Hex code or GUID - 分区类型代码，输入 `8e00` 即 `Linux LVM` 类型；

```
Command (? for help): n
Partition number (1-128, default 1):
First sector (34-1949261790, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-1949261790, default = 1949261790) or {+-}size{KMGTP}:
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 8e00
Changed type of partition to 'Linux LVM'

Command (? for help): 
```

输入 `w` 将变更写入磁盘：

```
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/md/getnas:0.
The operation has completed successfully.
```

操作完成，程序会自动退出。