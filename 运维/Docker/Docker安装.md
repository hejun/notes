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
    "data-root":"/hejun/data/docker"
  }
  EOF
  
  ```
  > `hosts` 是开始免密远程访问能力, 开启有风险，会增加服务器风险，仅限开发时使用

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

### 开启远程访问(非必须)

- 开启 `2375` 远程 `免密` 访问(增加漏洞风险，仅限开发时使用)
  
  [参考文档](https://docs.docker.com/engine/install/linux-postinstall/#configuring-remote-access-with-systemd-unit-file)

  - 修改 `systemd`
    
    ```
    vi /usr/lib/systemd/system/docker.service
    ```
    
    ```
    ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375 --containerd=/run/containerd/containerd.sock
    ```
    > 修改 ExecStart 添加 -H tcp://0.0.0.0:2375
    
    ```
    systemctl daemon-reload
    ```
    
    ```
    systemctl restart docker
    ```
    
  - 开启防火墙
    ```
    firewall-cmd --zone=public --add-port=2375/tcp --permanent
    ```
  
    ```
    firewall-cmd --reload
    ```

- 开启 `2376` 远程加密访问

  [参考文档](https://docs.docker.com/engine/security/protect-access/#use-tls-https-to-protect-the-docker-daemon-socket)

### Docker 访问代理

[参考文档](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)
