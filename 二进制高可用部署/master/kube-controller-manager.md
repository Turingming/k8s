## kube-controller-manager

```
cat > kube-controller-manager-csr.json << EOF
{
    "CN": "system:kube-controller-manager",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "hosts": [
      "127.0.0.1",
      "192.168.100.245",
      "192.168.100.246",
      "192.168.100.247"
    ],
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "k8s:kube-controller-manager",
        "OU": "system"
      }
    ]
}
EOF

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-controller-manager |cfssljson kube-controller-manager
```

```

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=https://192.168.100.252:8443 \
  --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager \
  --client-certificate=/etc/kubernetes/ssl/kube-controller-manager.pem \
  --client-key=/etc/kubernetes/ssl/kube-controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-context system:kube-controller-manager \
  --cluster=kubernetes \
  --user=system:kube-controller-manager \
  --kubeconfig=kube-controller-manager.kubeconfig

kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig


```

```
kube-controller-manager.service

[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/bin/kube-controller-manager \
  --kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
  --authentication-kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \
  --service-cluster-ip-range=10.254.0.0/16 \ #指定 Service Cluster IP 网段，必须和 kube-apiserver 中的同名参数一致
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --experimental-cluster-signing-duration=8760h \ #指定 TLS Bootstrap 证书的有效期；
  --root-ca-file=/etc/kubernetes/ssl/ca.pem \
  --service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem \
  --leader-elect=true \ #集群运行模式，启用选举功能；被选为 leader 的节点负责处理工作，其它节点为阻塞状态；
  --feature-gates=RotateKubeletServerCertificate=true \#开启 kublet server 证书的自动更新特性；
  --controllers=*,bootstrapsigner,tokencleaner \ #启用的控制器列表，tokencleaner 用于自动清理过期的 Bootstrap token；
  --horizontal-pod-autoscaler-use-rest-clients=true \
  --horizontal-pod-autoscaler-sync-period=10s \
  --tls-cert-file=/etc/kubernetes/ssl/kube-controller-manager.pem \
  --tls-private-key-file=/etc/kubernetes/ssl/kube-controller-manager-key.pem \
  --use-service-account-credentials=true \
  --alsologtostderr=true \
  --logtostderr=false \
  --log-dir=/var/log/kubernetes \
  --v=2
Restart=on
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target

```

```
启动服务

systemctl enable kube-controller-manager && systemctl start kube-controller-manager
```

```
查看当前的LADER
kubectl get endpoints kube-controller-manager --namespace=kube-system  -o yaml
```

