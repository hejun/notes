# Liux 防火墙

## 开启端口

```
firewall-cmd --zone=public --add-port=80/tcp --permanent  # --permanent永久生效，没有此参数重启后失效
```

# 重载入

```
firewall-cmd --reload
```

## 关闭端口

```
firewall-cmd --zone= public --remove-port=80/tcp --permanent
```

## 查看端口

```
firewall-cmd --zone= public --query-port=80/tcp
```

# More

[参考链接](https://www.cnblogs.com/moxiaoan/p/5683743.html)
