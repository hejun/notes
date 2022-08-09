## Docker安装

### 参考文档

[Docker Install Centos](https://docs.docker.com/engine/install/centos/)

### 安装

1. 卸载老版本

  ```sh
  yum remove docker \
             docker-client \
             docker-client-latest \
             docker-common \
             docker-latest \
             docker-latest-logrotate \
             docker-logrotate \
             docker-engine
  ```

2. 设置镜像仓库

  ```sh
  yum install yum-utils -y
  ```

  ```sh
  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```
  > 阿里云加速, 原生地址配置为 <br/> `yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

3. 安装

  ```sh
  yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
  ```

4. 设置镜像加速

  ```sh
  mkdir -p /etc/docker
  ```

  ```sh
  tee /etc/docker/daemon.json <<-'EOF'
  {
    "registry-mirrors": [
      "https://zv4800vv.mirror.aliyuncs.com",
      "http://f1361db2.m.daocloud.io",
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
    "data-root":"/hejun/data/docker"
  }
  EOF
  ```

  ```sh
  systemctl daemon-reload
  ```

- 启动服务

  ```sh
  systemctl enable docker
  ```

  ```sh
  systemctl start docker
  ```

### 扩展

- 开启 `2375` 远程 `免密` 访问(增加漏洞风险，仅限开发时使用)
  
  [参考文档](https://docs.docker.com/engine/install/linux-postinstall/#configuring-remote-access-with-systemd-unit-file)

  - 修改 `systemd`
    
    ```sh
    vi /usr/lib/systemd/system/docker.service
    ```
    
    ```sh
    ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375 --containerd=/run/containerd/containerd.sock
    ```
    > 修改 ExecStart 添加 -H tcp://0.0.0.0:2375
    
    ```sh
    systemctl daemon-reload
    ```
    
    ```sh
    systemctl restart docker
    ```
    
  - 开启防火墙
    ```sh
    firewall-cmd --zone=public --add-port=2375/tcp --permanent
    firewall-cmd --reload
    ```

- 开启 `2376` 远程加密访问

  [参考文档](https://docs.docker.com/engine/security/protect-access/#use-tls-https-to-protect-the-docker-daemon-socket)

- Docker 访问代理

  [参考文档](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)

  ```sh
  mkdir -p /etc/systemd/system/docker.service.d
  ```

  ```sh
  vi /usr/lib/systemd/system/docker.service
  ```

  ```sh
  [Service]
  Environment="HTTP_PROXY=${PROXY_HOST}"
  Environment="HTTPS_PROXY=${PROXY_HOST}"
  ```
  > `${PROXY_HOST}` 为代理地址

  ```sh
  systemctl daemon-reload
  ```

  ```sh
  systemctl restart docker
  ```
