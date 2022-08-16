## Ingress Nginx

参考文档

[IngressNginxController](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters)

### 安装

```sh
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/baremetal/deploy.yaml -O ingress-nginx.yaml
```

```sh
kubectl apply -f ingress-nginx.yaml
```

### 注意事项

- Image
  
  `ingress-nginx.yaml`的镜像带有 `***@sha256***`，需要去掉 @sha256 及之后的字段
