## Docker安装

### 参考文档

[Docker Install Centos](https://docs.docker.com/engine/install/centos/)

### 安装

- 卸载老版本
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

- 设置镜像仓库

  ```
  yum install yum-utils -y
  ```

  ```
  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```
  > 阿里云加速, 原生地址配置为 <br/> `yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

- 安装

  ```
  yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
  ```

- 设置镜像加速

  ```
  mkdir -p /etc/docker
  ```

  ```
  tee /etc/docker/daemon.json <<-'EOF'
  {
    "registry-mirrors": [
      "https://zv4800vv.mirror.aliyuncs.com",
      "https://docker.mirrors.ustc.edu.cn",
      "https://hub-mirror.c.163.com"
    ],
    "exec-opts": [
      "native.cgroupdriver=systemd"
    ],
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
  ```

  ```
  systemctl daemon-reload
  ```

- 启动服务

  ```
  systemctl enable docker
  ```

  ```
  systemctl start docker
  ```

### 开启远程访问 (非必须,且增加安全隐患,开发时使用)

- 开启2375无密访问
  ```
  vi /usr/lib/systemd/system/docker.service
  ```

  ```
  ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock
  ```
  > 修改 `ExecStart` 添加 `-H tcp://0.0.0.0:2375`

  ```
  systemctl daemon-reload
  ```

  ```
  systemctl restart docker
  ```

  ```
  firewall-cmd --zone=public --add-port=2375/tcp --permanent
  ```

  ```
  firewall-cmd --reload
  ```

- 开启2376加密访问

  [参考文档](https://docs.docker.com/engine/security/protect-access/#use-tls-https-to-protect-the-docker-daemon-socket)
