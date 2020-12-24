## Linux SWAP 分区

> 添加SWAP分区

```
dd if=/dev/zero of=/mnt/swap bs=1M count=1024

# bs、count大小可以自定义，比如bs=1M count=1024代表设置1G大小SWAP分区。
```

```
mkswap /mnt/swap
```

```
swapon /mnt/swap
```

```
/mnt/swap swap swap defaults 0 0
```

```
vi /etc/sysctl.conf

vm.swappiness = 30
```

> 关闭SWAP分区

```
# 查询SWAP分区设置

free -m
```

```
# [$SWAP_File]SWAP分区标识。

swapoff [$SWAP_File]
```

```
vi /etc/fstab

# 删除或注释相关配置，取消SWAP的自动挂载
```

```
vi /etc/sysctl.conf

vm.swappiness = 0
```