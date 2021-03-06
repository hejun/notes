# Docker安装配置

## 安装

参考: [Get Docker](https://docs.docker.com/engine/install/centos/)

### 1.卸载老版本

```
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

### 2.设置国内镜像仓库

```
yum install -y yum-utils

yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

```
# 上述有时候会失效,可更改 docker-ce.repo 的 baseurl 为下面这个
# vi /etc/yum.repos.d/repo.repo

https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/7/x86_64/stable/
```

### 3.安装

```
yum install docker-ce docker-ce-cli containerd.io -y
```

### 4.设置镜像加速

```
mkdir -p /etc/docker

tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://zv4800vv.mirror.aliyuncs.com", "https://docker.mirrors.ustc.edu.cn"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts":{
    "max-size": "100m",
    "max-file": "1"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "graph":"/hejun/data/docker"
}
EOF

systemctl daemon-reload
```

### 5.设置开机自启,启动

```
systemctl enable docker

systemctl start docker
```

## 开启远程访问

- 开启2375无密访问

修改 docker.service

```
# vi /usr/lib/systemd/system/docker.service

[Service]
# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock
```

重启

```
systemctl daemon-reload

systemctl restart docker
```

- 开启2376加密访问

参考: [Docker TLS](https://docs.docker.com/engine/security/https/)

1. 创建 CA 证书

```
openssl genrsa -aes256 -out ca-key.pem 4096

# 根据提示输入密码
```

```
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem

# 根据提示输入证书信息
```

```
openssl genrsa -out server-key.pem 4096
```

```
# 替换 $HOST 为本机IP

openssl req -subj "/CN=$HOST" -sha256 -new -key server-key.pem -out server.csr
```

```
# 替换 $HOST 为本机IP

echo subjectAltName = DNS:$HOST,IP:127.0.0.1 >> extfile.cnf
```

```
echo extendedKeyUsage = serverAuth >> extfile.cnf
```

```
openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem -extfile extfile.cnf
```

```
openssl genrsa -out key.pem 4096
```

```
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
```

```
echo extendedKeyUsage = clientAuth > extfile-client.cnf
```

```
openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -CAcreateserial -out cert.pem -extfile extfile-client.cnf
```

```
rm -v client.csr server.csr extfile.cnf extfile-client.cnf
```

```
chmod -v 0400 ca-key.pem key.pem server-key.pem
```

```
chmod -v 0444 ca.pem server-cert.pem cert.pem
```

2. 修改 docker.service

```
# vi /usr/lib/systemd/system/docker.service

# ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=ca.pem --tlscert=server-cert.pem --tlskey=server-key.pem -H=0.0.0.0:2376 -H fd:// --containerd=/run/containerd/containerd.sock
```

3. 设置默认访问证书

```
mkdir -pv ~/.docker

cp -v {ca,cert,key}.pem ~/.docker

vi /etc/profile
export DOCKER_HOST=tcp://127.0.0.1:2376 DOCKER_TLS_VERIFY=1

source /etc/profile
```

## 开通防火墙端口

- 开通2375

```
firewall-cmd --zone=public --add-port=2375/tcp --permanent

firewall-cmd --reload
```

- 开通2376

```
firewall-cmd --zone=public --add-port=2376/tcp --permanent

firewall-cmd --reload
```
