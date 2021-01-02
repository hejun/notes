# 配置 Let's encrypt 证书

1. 获取Certbot

```
wget https://dl.eff.org/certbot-auto

chmod u+x certbot-auto
```

2. 申请证书

```
./certbot-auto certonly  -d "*.hejun.com" -d "hejun.com" --manual --preferred-challenges dns-01  --server https://acme-v02.api.letsencrypt.org/directory
```

参数说明:

- certonly，表示安装模式，Certbot 有安装模式和验证模式两种类型的插件。
- --manual，表示手动安装插件，Certbot 有很多插件，不同的插件都可以申请证书，用户可以根据需要自行选择。
- -d，为哪些主机申请证书，如果是通配符，输入 *.hubinqiang.com（替换为自己的域名）。
- --preferred-challenges，使用 DNS 方式校验域名所有权。
- --server，Let’s Encrypt ACME v2 版本使用的服务器不同于 v1 版本，需要显示指定。

在安装过程中需要根据要求添加 `_acme-challenge.hubinqiang.com` TXT域名验证

3. Nginx配置证书

```
server {
    server_name hejun.com www.hejun.com;
    listen 443 http2 ssl;

    ssl_certificate /etc/letsencrypt/live/hubinqiang.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/hubinqiang.com/privkey.pem;
    ssl_trusted_certificate  /etc/letsencrypt/live/hubinqiang.com/chain.pem;

    # ... 其他代码
}
```

4. 证书更新

```
./certbot-auto renew
```