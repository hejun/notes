## Calico网络

参考文档

- [Calico安装](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises)
- [安装前置条件](https://projectcalico.docs.tigera.io/getting-started/kubernetes/requirements)

### 安装

- 环境准备

  1. 允许 `Calico` 管理 `NetworkManager` 接口

     ```sh
     vi /etc/NetworkManager/conf.d/calico.conf
     ```
  
     ```sh
     [keyfile]
     unmanaged-devices=interface-name:cali*;interface-name:tunl*;interface-name:vxlan.calico;interface-name:wireguard.cali
     ```

 2. 开启端口

    - 控制端

      ```sh
      firewall-cmd --zone=public --add-port=179/tcp --add-port=443/tcp --add-port=6443/tcp --add-port=4789/tcp --permanent
      firewall-cmd --reload
      ```

    - 工作节点

      ```sh
      firewall-cmd --zone=public --add-port=179/tcp --add-port=4789/tcp --permanent
      firewall-cmd --reload
      ```

- 安装

  ```sh
  wget https://projectcalico.docs.tigera.io/manifests/calico.yaml
  ```

  ```sh
  kubectl apply -f calico.yaml
  ```

### 注意事项

- Pod网段

  ```sh
  - name: CALICO_IPV4POOL_CIDR
    value: "10.244.0.0/16"
  ```
  > `Calico` 默认网段为 `192.168.0.0/16`, 这个值是 `Kubenetes` 初始化时配置的 `pod-network-cidr`,应保持一致

- 多网卡

  ```sh
  - name: IP_AUTODETECTION_METHOD
    value: "interface=enp0s3"
  ```
  > 多网卡需要指定通信网卡, 在 `containers` 的 `name: calico-node` 容器下配置, `enp0s3` 是网卡名称，需要改为实际名称
  > [参考资料](https://projectcalico.docs.tigera.io/reference/node/configuration#configuring-bgp-networking)
