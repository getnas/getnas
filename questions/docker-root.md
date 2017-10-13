# 修改 Docker 默认存储路径

## 查看 Docker 的默认存储路径

```
# docker info
```

> 通常，默认存储路径为 `/var/lib/docker`

## 停止 docker 服务

```
# systemctl stop docker
```

## 创建 Drop-In 文件

Drop-In 文件能够覆盖 docker 默认服务配置文件 `/lib/systemd/system/docker.service` 的参数。

### 第一步 创建 docker.service.d 目录

```
# mkdir /etc/systemd/system/docker.service.d
```

### 第二步 创建 docker.conf 文件

```
# nano /etc/systemd/system/docker.service.d/docker.conf
```

添加以下配置信息，将 `/mnt/new_volume` 替换为最终作为 docker 存储的实际路径：

```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --graph="/mnt/new_volume" --storage-driver=devicemapper 
```

## 重载 systemd 守护进程

```
# systemctl daemon-reload 
```

## 重启 docker 服务

```
# systemdctl start docker
```

***Note - Existing Containers and Images***  
If you already have containers or images in `/var/lib/docker` you may wish to stop and back these up before moving them to the new root location. Moving can be done by either `rsync -a /var/lib/docker/* /path/to/new/root` or if permissions do not matter, you can simply use mv  or cp too.

参考：[https://sanenthusiast.com/change-default-image-container-location-docker/](https://sanenthusiast.com/change-default-image-container-location-docker/)

