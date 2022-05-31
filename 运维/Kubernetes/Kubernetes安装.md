## Kubernetes安装

[参考文档](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/)

### 安装

- 前置条件

    - CPU 2核+
    - 内存 2G+

- 安装 `Containerd`
  > `Docker` 自带 `Containerd`, 如果已经安装，可跳过

  ```
  yum install yum-utils -y
  ```
  
  ```
  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  ```
  > 阿里云加速, 原生地址配置为 <br/> `yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

  ```
  yum install containerd.io -y
  ```
  
  ```
  containerd config default > /etc/containerd/config.toml
  ```
  > 导出并覆盖原本配置文件 (可修改配置文件中 `root` 来更改默认存储位置)

  ```
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
  ```
  > 在配置文件中找到上述配置, 修改 `SystemdCgroup` 为 `true`

  ```
  cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
  overlay
  br_netfilter
  EOF
  
  modprobe overlay
  modprobe br_netfilter
  
  cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
  net.bridge.bridge-nf-call-iptables  = 1
  net.ipv4.ip_forward                 = 1
  net.bridge.bridge-nf-call-ip6tables = 1
  EOF
  
  sysctl --system
  
  ```
  
  ```
  systemctl enable --now containerd
  ```
  > 设置自启并启动

- `Containerd` 代理
  > 非必须步奏

  ```
  vi /usr/lib/systemd/system/containerd.service
  ```
  
  ```
  [Service]
  Environment="HTTP_PROXY=${PROXY_URL}"
  Environment="HTTPS_PROXY=${PROXY_URL}"
  Environment="NO_PROXY=${NO_PROXY_UTL}"
  ```
  > 如示添加代理, `${PROXY_URL}` 为代理地址, `${NO_PROXY_UTL}` 为不代理的地址,逗号分割

  ```
  systemctl daemon-reload
  ```
  
  ```
  systemctl restart containerd
  ```

- 初始化 `Kubernetes` 环境

  - 设置 `hosts`

    ```
    vi /etc/hosts
    ```
    > 然后添加 ip 与本机的映射
  
  - 关闭SWAP分区
    ```
    swapoff -a
    ```

    ```
    vi /etc/fstab
    ```
    > 然后注释掉SWAP的自动挂载

  - 关闭防火墙
    ```
    system stop firewalld
    systemctl disable firewalld
    
    ```
    > 不想关闭防火墙的可以参考官方端口文档(不推荐) [端口](https://kubernetes.io/zh/docs/reference/ports-and-protocols/)

  - 关闭SELinux

    ```
    sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    setenforce 0
      
    ```

  - 重置 iptables

    ```
    iptables -F
    iptables -F -t nat
    iptables -X
    iptables -X -t nat
    iptables -P FORWARD ACCEPT
      
    ```

  - 允许 iptables 检查桥接流量

    ```
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    br_netfilter
    EOF

    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF

    sysctl --system
    
    ```

- 安装 `kubernetes` 相关组件

    - 设置 `kubernetes` 国内源

      ```
      cat <<EOF > /etc/yum.repos.d/kubernetes.repo
      [kubernetes]
      name=Kubernetes
      baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
      enabled=1
      gpgcheck=0
      repo_gpgcheck=0
      gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
      EOF

      ```
      > 参考 `https://developer.aliyun.com/mirror/kubernetes`

- 安装
  ```
  yum install kubelet kubeadm kubectl --disableexcludes=kubernetes -y 
  ```

  ```
  crictl config --set runtime-endpoint=unix:///run/containerd/containerd.sock
  crictl config --set image-endpoint=unix:///run/containerd/containerd.sock
  
  ```
  > `crictl` 默认使用 `dockershim`, 需要修改为 `containerd`

  ```
  systemctl enable --now kubelet
  ```

- 使用 kubeadm 创建集群

    - 下载镜像

      ```
      kubeadm config images pull
      ```
      > 国内基本下不下来, 可通过上一步代理尝试下载, 也可通过下载的镜像进行导入导出<br/> 导出命令: `ctr -n=k8s.io image export - ${IMAGE:TAG} > ${TAR_NAME}.tar`,<br/> 导入命令: `ls *.tar | xargs -i ctr -n=k8s.io image import {}`

    - 引导 `Master` 启动

      ```
      kubeadm init \
      --apiserver-advertise-address=${MASTER_IP} \
      --pod-network-cidr=10.244.0.0/16
      ```

    - 引导 `Node` 启动
      > 在每个Node节点粘贴执行引导 Master 后输出的 <br/> `kubeadm join **** --token ****` 语句

    - 设置 kubectl 工具

      - `root` 用户
  
        ```
        echo "# Kubectl" >> /etc/profile
        echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
        ```
  
        ```
        source /etc/profile
        ```

      - 非`root` 用户

        ```
        mkdir -p $HOME/.kube
        cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        chown $(id -u):$(id -g) $HOME/.kube/config
        ```

      - 验证是否成功

        ```
        kubectl get nodes
        ```
        > 有结果即表示成功

### 添加额外的访问IP

- 删除当前kubernetes集群下的 `apiserver` 的 `cert` 和 `key`

  ```
  rm -rf /etc/kubernetes/pki/apiserver.*
  ```

- 生成新的 `apiserver` 的 `cert` 和 `key`

  ```
  kubeadm init phase certs apiserver --apiserver-advertise-address ${APISERVER_HOST} --apiserver-cert-extra-sans ${额外的IP}
  ```

- 刷新 `admin.conf`

  ```
  kubeadm alpha certs renew admin.conf
  ```
