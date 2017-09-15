# 调整系统 U 盘空间

安装系统时我们删除了 `Swap` 交换分区，U 盘因此空出部分空间。可以将这部分空间扩展给系统分区来提高可用空间。

## 准备工作

完成后续步骤需使用 `parted` 分区工具，如果尚未安装：

```
getnas@getnas:~$ sudo apt install parted
```

## 调整分区

### 第一步 交互模式打开 parted

```
getnas@getnas:~$ sudo parted /dev/sda
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

### 第二步 查看分区信息

使用 `p` 或 `print` 命令查看分区信息：

```
(parted) p
Model: SanDisk Cruzer Fit (scsi)
Disk /dev/sda: 8004MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  6931MB  6930MB  primary  ext4         boot
```

输出结果显示，U 盘设备 `/dev/sda` 容量为 `8004MB`，存在一个编号为 `1` 的分区占用空间为 `6930MB`，即系统所在分区。

### 第三步 调整分区容量

使用 `resizepart` 命令将分区容量调整至最大：

* Partition number - 分区编号，填写 `1`；
* Yes/No - 提示分区正在使用，是否继续操作，输入 `y` 或 `yes` 继续；
* End - 设置分区最终容量，输入 `-1` 代表设备最末端，即设备所有空间；

```
(parted) resizepart
Partition number? 1
Warning: Partition /dev/sda1 is being used. Are you sure you want to continue?
Yes/No? y
End?  [6931MB]? -1
```

再次使用 `p` 或 `print` 命令，可以看到 `1` 号分区的容量已经被扩展至最大：

```
(parted) p
Model: SanDisk Cruzer Fit (scsi)
Disk /dev/sda: 8004MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  8003MB  8002MB  primary  ext4         boot
```

## 调整文件系统

使用 `resize2fs` 命令调整系统分区的文件系统以适配分区扩展后的容量：

```
getnas@getnas:~$ sudo resize2fs /dev/sda1
resize2fs 1.43.4 (31-Jan-2017)
/dev/sda1 上的文件系统已被挂载于 /；需要进行在线调整大小

old_desc_blocks = 1, new_desc_blocks = 1
/dev/sda1 上的文件系统现在为 1953676 个块（每块 4k）。
```

使用 `fdisk` 命令查看调整以后的分区情况：

```
getnas@getnas:~$ sudo fdisk -l
Disk /dev/sda: 7.5 GiB, 8004304896 bytes, 15633408 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe4ff1260

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1  *     2048 15631455 15629408  7.5G 83 Linux
```

可以看到，系统分区 `/dev/sda1` 的容量与其所在的 U 盘设备 `/dev/sda` 容量相同了。