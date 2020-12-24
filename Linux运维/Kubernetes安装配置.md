## Kubernetes

> 搭建前提

- CPU 2核+
- 内存 2G+

> 基于 `Kubeadm` 安装

### 1. 初始化系统

1. 关闭SWAP分区

   ```
   # [$SWAP_File]SWAP分区标识。

   swapoff [$SWAP_File]
   ```

   ```
   vi /etc/fstab

   # 删除或注释相关配置，取消SWAP的自动挂载
   ```

2. 开放所需端口

   - Master节点

     ```
     firewall-cmd --zone=public --add-port=6443/tcp --permanent

     firewall-cmd --zone=public --add-port=2379-2380/tcp --permanent

     firewall-cmd --zone=public --add-port=10250-10252/tcp --permanent

     firewall-cmd --reload
     ```

   - Node节点

     ```
     firewall-cmd --zone=public --add-port=10250/tcp --permanent

     firewall-cmd --zone=public --add-port=30000-32767/tcp --permanent

     firewall-cmd --reload
     ```

3. 关闭SELinux

   ```
   setenforce 0

   sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
   ```

4. 将桥接的IPV4流量传递到ipatbles的链

   ```
   cat <<EOF >  /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF

   sysctl --system
   ```

### 2. 安装 `Docker`

安装方法参考笔记 [Docker安装配置](https://github.com/hejun/notes/blob/master/Linux%E8%BF%90%E7%BB%B4/Docker%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AE.md)

注意: 版本请参考 [容器运行时](https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/#docker) 以免版本不兼容

### 3. 安装 `kubeadm` `kubelet` `kubectl`

1. 设置kubernetes国内源

   ```
   cat <<EOF > /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   EOF
   ```

2. 安装 `kubeadm` `kubelet` `kubectl`

   ```
   yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

   systemctl enable --now kubelet
   ```

### 4. 使用 `kubeadm` 引导集群

1. 引导Master

   ```
   kubeadm init \
     --apiserver-advertise-address=192.168.1.5 \
     --image-repository registry.aliyuncs.com/google_containers \
     --pod-network-cidr=10.244.0.0/16
   ```

   注: [kubeadm参数说明](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/)