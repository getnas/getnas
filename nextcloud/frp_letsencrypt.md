# 使用 Let's Encrypt 实现加密的 HTTPS 协议

我们已经使用 frp 反向代理工具实现了本地 NAS 服务器的互联网访问，出于数据传输的安全性考虑，有必要启用 `https` 加密协议。

## 安装 certbot

```
$ sudo apt-get install certbot 
```

