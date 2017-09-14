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


