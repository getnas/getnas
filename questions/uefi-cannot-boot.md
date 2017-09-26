# UEFI 模式 Debian 启动失败进入 initramfs

## 系统版本和环境

Debian 4.9.30-2+deb9u5 (2017-09-19) x86_64

采用 UEFI 模式安装 Debian 9 到 U 盘 `/dev/sda`，分区：

* /dev/sda1, fat, EFI, 100mb
* /dev/sda2, ext4, /boot, 200mb
* /dev/sda3, ext4, /, 所有剩余空间

## 问题回溯

不接其他硬盘可以正常启动。接入其他硬盘启动系统时会报错 `ACPI Error: Namespace lookup failure`，随后系统会提示无法找到 `/dev/sda3` 系统分区启动失败并出现 `initramfs` 提示符。

## 解决办法

`ACPI Error` 报错与找不到 `/dev/sda3` 分区完全是两个问题。前者是 Linux 4.9 内核的问题（机制），后者是 `grub2` 的问题。

能看到上述信息，说明 UEFI 模式引导系统启动的工作是正常的。

系统无法正常启动主要是因为 `grub2` 的配置文件中使用了分区的 `UUID` 值，而 UEFI 模式下的 `grub2` 是无法正常识别 `UUID` 值的。

只需将 `/boot/grub/grub.cfg` 配置文件中 `/dev/sda3` 分区的对应的所有 `UUID` 值替换为它的 `PARTUUID` 值即可。

### 第一步 查看系统分区的 PARTUUID 值

```
getnas@getnas:~$ sudo blkid
/dev/sda1: UUID="A219-87C8" TYPE="vfat" PARTUUID="2e7ff1b0-9d63-45aa-bf49-b54c02fb142e"
/dev/sda2: UUID="f8c8d331-4df2-416e-a6ca-1371bf8cf84d" TYPE="ext4" PARTUUID="8c1d7e31-c87e-4727-98ef-0f61dae0af3c"
/dev/sda3: UUID="9d6d48c5-9f5e-4f6c-b252-efb83ee167ae" TYPE="ext4" PARTUUID="eb50b818-4b17-4578-92d7-5673ae1898cd"
```

最后一条记录中可以看到 `/dev/sda3` 分区的 `UUID` 和 `PARTUUID`。

### 第二步 编辑 grub2 配置文件

在编辑之前，建议备份 grub2 配置文件：

```
getnas@getnas:~$ sudo cp /boot/grub/grub.cfg /boot/grub/grub.cfg_bak
```

使用 `nano` 编辑器打开配置文件：

```
getnas@getnas:~$ sudo nano /boot/grub/grub.cfg
```

在编辑器中使用`替换功能`组合键 `CTRL + \`，根据 `nano` 编辑器的提示搜索 `/dev/sda1` 分区的 `UUID` 并用 `PARTUUID` 进行全部替换。

编辑完成后使用组合键 `CTRL + o` 保存，使用 `CTRL + x` 退出编辑器。

### 第三步 更新 grub2 让新配置生效

```
herald@homeserver:~$ sudo update-grub
```

## 总结

解决此问题的过程中，屏幕提示信息对笔者产生了很大的干扰，始终以为 `ACPI` 错误与系统无法启动有关系，而实际上它们完全是两个不同的问题。