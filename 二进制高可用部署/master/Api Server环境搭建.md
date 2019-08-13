## Api Server环境搭建

```
生成kubernetes 证书

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "192.168.100.245",
	"192.168.100.246",
	"192.168.100.247",
	"192.168.100.248",
	"192.168.100.249",
	"192.168.100.250",
	"192.168.100.251",
	"192.168.100.252",
	"192.168.100.253",
	"10.254.0.1",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
master主机ip
服务ip

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kubernetes-csr.json |cfssljson -bare kubernetes

```

```
 TLS Bootstrapping 使用的Token，可以使用命令 head -c 16 /dev/urandom | od -An -t x | tr -d ' ' 生成
 cat token.csv 
2291410a69d70b168e3f3b12055dfbec,kubelet-bootstrap,10001,"system:kubelet-bootstrap"
```

```
配置Apiserver启动参数
 cat kube-apiserver
KUBE_APISERVER_OPTS="--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
     --advertise-address=192.168.100.245 \   #多个master部署需要更换IP
     --bind-address=192.168.100.245 \        #多个master部署需要更换IP
     --insecure-bind-address=192.168.100.245 \ #多个master部署需要更换IP
     --authorization-mode=RBAC,Node \
     --kubelet-https=true \
     --enable-bootstrap-token-auth \   #启用 kubelet bootstrap 的 token 认证；
     --token-auth-file=/etc/kubernetes/token.csv \
     --service-cluster-ip-range=10.254.0.0/16 \
     --service-node-port-range=30000-32766 \
     --tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem \
     --tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
     --client-ca-file=/etc/kubernetes/ssl/ca.pem \
     --service-account-key-file=/etc/kubernetes/ssl/ca-key.pem \
     --etcd-cafile=/etc/kubernetes/ssl/ca.pem \
     --etcd-certfile=/etc/kubernetes/ssl/kubernetes.pem \
     --etcd-keyfile=/etc/kubernetes/ssl/kubernetes-key.pem \
     --etcd-servers=https://192.168.100.245:2379,https://192.168.100.246:2379,https://192.168.100.247:2379 \
     --enable-swagger-ui=true \
     --allow-privileged=true \
     --apiserver-count=3 \      #指定集群运行模式，多台 kube-apiserver 会通过 leader选举产生一个工作节点，其它节点处于阻塞状态；
     --audit-log-maxage=30 \
     --audit-log-maxbackup=3 \
     --audit-log-maxsize=100 \
     --audit-log-path=/var/lib/audit.log \
     --event-ttl=1h \
     --logtostderr=true \
     --v=6" 
```

```
配置Apiserver服务调用apiserver启动参数

cat kube-apiserver.service 
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
EnvironmentFile=-/etc/systemd/system/kube-apiserver
ExecStart=/usr/bin/kube-apiserver $KUBE_APISERVER_OPTS 
Restart=on-failure
RestartSec=5
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target

```

```
启动服务

systemctl enable kube-apiserver && systemctl start kube-apiserver
```

```
验证服务

kubectl cluster-info
```

