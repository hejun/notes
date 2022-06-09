## Calico网络

[参考文档](https://projectcalico.docs.tigera.io/getting-started/kubernetes/self-managed-onprem/onpremises)

### 安装

```
wget https://projectcalico.docs.tigera.io/manifests/calico.yaml
```

```
kubectl apply -f calico.yaml
```

### 注意事项

- 多网卡

  > [参考资料](https://projectcalico.docs.tigera.io/reference/node/configuration#configuring-bgp-networking)

  ```
  - name: IP_AUTODETECTION_METHOD
    value: "interface=enp0s3"
  ```

  > 多网卡需要指定通信网卡, 在 `containers` 的 `name: calico-node` 容器下配置, `enp0s3` 是网卡名称，需要改为实际名称

- 防火墙

  > [参考资料](https://projectcalico.docs.tigera.io/getting-started/kubernetes/requirements)

  > `calico` 最好是关闭防火墙，否则需要按文档配置处理