# Kubernetes

## 搭建前提

- CPU 2核+
- 内存 2G+

## 基于 `Kubeadm` 安装

参考 [使用 kubeadm 创建集群](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

### 1. 初始化系统

1. 关闭SWAP分区

   ```
   swapoff -a
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
   
   使用 `kubectl` 工具
   
   - 非 `root` 用户

      ```
      mkdir -p $HOME/.kube
      
      cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      
      chown $(id -u):$(id -g) $HOME/.kube/config
      ```

   - `root` 用户

      ```
      # vi /etc/profile
     
      export KUBECONFIG=/etc/kubernetes/admin.conf
      
      source /etc/profile
      ```

   验证方式

   ```
   kubectl get nodes
   ```

2. 引导Node

   ```
   # 下列语句并不对,需要Copy引导Master时提示的类似于下列的语句
   
   kubeadm join 192.168.1.5:6443 --token ffr7uf.s3auu9n7bomks29x \
       --discovery-token-ca-cert-hash sha256:256114a4aa95cf044a01c944ae7cf14e67dc9c2cbcca5eb496ebe082c1028e32
   ```

3. 安装Pod网络附加组件

   由于网络原因,访问 [Flannel](https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml)
   将文件内容放置本地,之后执行下列命令
   
   ```
   # 注: ... 为上述文件绝对路径

   kubectl apply -f ...
   ```
   
   执行下列命令查看进度,等待所有完成
   
   ```
   kubectl get pods -n kube-system
   ```

## 基于二进制方式安装
