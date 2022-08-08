## Kubernetes安装

[参考文档](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/)

### 安装

- 前置条件
  - CPU 2核+
  - 内存 2G+
  - CentOS 7+

- 环境准备
  1. 关闭 `selinux`

     ```sh
     sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
     setenforce 0
     ```

  2. 禁用交换分区

     ```sh
     swapoff -a
     ```
     ```sh
     vi /etc/fstab
     ```
     然后注释掉所有类型为swap的交换分区

  3. 重置 `iptables`

     ```sh
     iptables -F
     iptables -F -t nat
     iptables -X
     iptables -X -t nat
     iptables -P FORWARD ACCEPT
     ```

  4. 允许 iptables 检查桥接流量

     ```sh
     cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
     br_netfilter
     EOF

     cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
     net.bridge.bridge-nf-call-ip6tables = 1
     net.bridge.bridge-nf-call-iptables = 1
     EOF
     sudo sysctl --system
     ```

  5. 开启必要端口

     - 控制端

       ```sh
       firewall-cmd --zone=public \
                    --add-port=2379-2380/tcp \
                    --add-port=6443/tcp \
                    --add-port=10250/tcp \
                    --add-port=10257/tcp \
                    --add-port=10259/tcp --permanent
       firewall-cmd --reload
       ```

     - 工作节点

       ```sh
       firewall-cmd --zone=public --add-port=10250/tcp --add-port=30000-32767/tcp --permanent
       firewall-cmd --reload
       ```

- 安装 `runtime`

  [Docker 安装](../Docker/Docker安装.md)

- 安装 `kubeadm` `kubelet` 和 `kubectl`

  ```sh
  cat <<EOF > /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
  EOF
  ```
  > 参考 `https://developer.aliyun.com/mirror/kubernetes`

  ```sh
  yum list kubeadm --showduplicates --nogpgcheck -y | sort -r
  ```
  > 查询 `kubeadm` 版本，非必须

  ```sh
  yum install -y --nogpgcheck kubelet kubeadm kubectl --disableexcludes=kubernetes
  ```
  > 指定安装版本<br/>`yum install -y --nogpgcheck kubelet-${VERSION} kubeadm-${VERSION} kubectl-${VERSION} --disableexcludes=kubernetes`

  ```sh
  systemctl enable --now kubelet
  ```

- 引导集群

  1. 查看所需镜像
  
     ```sh
     kubeadm config images list
     ```
  2. 拉取镜像

     ```sh
     kubeadm config images pull
     ```
     > 需能访问 `k8s.gcr.io`

  3. 引导控制端

     ```sh
     kubeadm init \
       --kubernetes-version=${KUBENETES_VERSION} \
       --apiserver-advertise-address=${MASTER_IP} \
       --pod-network-cidr=10.244.0.0/16
     ```

  4. 引导工作节点

     在每个Node节点粘贴执行控制端输出的类似下方的语句:
     
     `kubeadm join **** --token ****`

  5. 设置 `kubectl` 工具

     - `root` 用户
     
       ```sh
       echo "# Kubectl" >> /etc/profile
       echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
       ```
  
       ```sh
       source /etc/profile
       ```

     - 非 `root` 用户

        ```sh
        mkdir -p $HOME/.kube
        cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        chown $(id -u):$(id -g) $HOME/.kube/config
        ```

  6. 验证

     ```sh
     kubectl get nodes
     ```

### 扩展

- 添加额外的访问IP

  1. 删除当前kubernetes集群下的 `apiserver` 的 `cert` 和 `key`

     ```sh
     rm -rf /etc/kubernetes/pki/apiserver.*
     ```

  2. 生成新的 `apiserver` 的 `cert` 和 `key`

     ```sh
     kubeadm init phase certs apiserver --apiserver-advertise-address ${APISERVER_HOST} --apiserver-cert-extra-sans ${额外的IP}
     ```

  3. 刷新 `admin.conf`

     ```sh
     kubeadm alpha certs renew admin.conf
     ```
