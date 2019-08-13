## 部署Flannel

```
cat > flanneld-csr.json <<EOF
{
  "CN": "flanneld",
  "hosts": [],
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

cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes flanneld-csr.json | cfssljson -bare flanneld
```

```
etcdctl \
  --endpoints=https://192.168.100.245:2379,https://192.168.100.246:2379,https://192.168.100.247:2379 \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/flanneld.pem \
  --key-file=/etc/kubernetes/ssl/flanneld-key.pem \
  set /kubernetes/network/config '{"Network":"172.30.0.0/16", "SubnetLen": 24, "Backend": {"Type": "vxlan"}}'
```

```
 mkdir -p  /run/flannel/docker

cat flanneld.service 
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
ExecStart=/usr/bin/flanneld \
  -etcd-cafile=/etc/kubernetes/ssl/ca.pem \
  -etcd-certfile=/etc/kubernetes/ssl/flanneld.pem \
  -etcd-keyfile=/etc/kubernetes/ssl/flanneld-key.pem \
  -etcd-endpoints=https://192.168.100.245:2379,https://192.168.100.246:2379,https://192.168.100.247:2379 \
  -etcd-prefix=/kubernetes/network
ExecStartPost=/usr/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=on-failure

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service



```

```
配置docker支持flannel

cat /etc/systemd/system/multi-user.target.wants/docker.service 
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
EnvironmentFile=/run/flannel/docker                  #添加环境变量
ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS   #调用参数
ExecReload=/bin/kill -s HUP $MAINPID
# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
# Uncomment TasksMax if your systemd version supports it.
# Only systemd 226 and above support this version.
#TasksMax=infinity
TimeoutStartSec=0
# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes
# kill only the docker process, not all processes in the cgroup
KillMode=process
# restart the docker process if it exits prematurely
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target

```

