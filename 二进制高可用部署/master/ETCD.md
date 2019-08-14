## ETCD环境搭建

```
下载安装包
下载地址
https://github.com/etcd-io/etcd/releases/tag/v3.3.13

保存成脚本
ETCD_VER=v3.3.13
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}
curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
如果不行，使用下面链接直接下载
https://github.com/etcd-io/etcd/releases/download/v3.3.13/etcd-v3.3.13-linux-amd64.tar.gz

```

```
节点安装

NODE_IPS=("192.168.100.245" "192.168.100.246" "192.168.100.247")

各节点安装脚本
for node_ip in ${NODE_IPS[@]};do
        echo ">>> ${node_ip}"
        scp /sa/soft/etcd-v3.3.13-linux-amd64/etcd* root@${node_ip}:/usr/bin
        ssh root@${node_ip} "chmod +x /usr/bin/etcd*"
        ssh root@${node_ip} "mkdir -p /etcd/cert"
        ssh root@${node_ip} "mkdir -p /etcd/data"
        scp /etcd/cert/* root@${node_ip}:/etcd/cert
done
```

```
创建配置文件模板

cat > /etcd/etcd.service.template <<EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos
[Service]
User=root
Type=notify
WorkingDirectory=/etcd/data/
ExecStart=/usr/bin/etcd \
    --data-dir=/etcd/data/ \
    --name ##NODE_NAME## \
    --cert-file=/etcd/cert/etcd.pem \
    --key-file=/etcd/cert/etcd-key.pem \
    --trusted-ca-file=/etcd/cert/ca.pem \
    --peer-cert-file=/etcd/cert/etcd.pem \
    --peer-key-file=/etcd/cert/etcd-key.pem \
    --peer-trusted-ca-file=/etcd/cert/ca.pem \
    --peer-client-cert-auth \
    --client-cert-auth \
    --listen-peer-urls=https://##NODE_IP##:2380 \
    --initial-advertise-peer-urls=https://##NODE_IP##:2380 \
    --listen-client-urls=https://##NODE_IP##:2379,http://127.0.0.1:2379\
    --advertise-client-urls=https://##NODE_IP##:2379 \
    --initial-cluster-token=etcd-cluster-0 \
    --initial-cluster=etcd0=https://192.168.100.245:2380,etcd1=https://192.168.100.246:2380,etcd2=https://192.168.100.247:2380 \
    --initial-cluster-state=new
Restart=on-failure
RestartSec=5
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

```

```
配置各节点

生成环境变量
NODE_NAMES=("etcd0" "etcd1" "etcd2")
NODE_IPS=("192.168.100.245" "192.168.100.246" "192.168.100.247")
生成配置文件
for (( i=0; i < 3; i++ ));do
        sed -e "s/##NODE_NAME##/${NODE_NAMES[i]}/g" -e "s/##NODE_IP##/${NODE_IPS[i]}/g" /etcd/etcd.service.template > /etcd/etcd-${NODE_IPS[i]}.service
done
分发各节点
for node_ip in ${NODE_IPS[@]};do
        echo ">>> ${node_ip}"
        scp /etcd/etcd-${node_ip}.service root@${node_ip}:/etc/systemd/system/etcd.service
done

```

```
启动服务

启动各节点服务
for node_ip in ${NODE_IPS[@]};do
        echo ">>> ${node_ip}"
        ssh root@${node_ip} "systemctl daemon-reload && systemctl enable etcd && systemctl start etcd"
done

```

```
健康检查

查看状态
for node_ip in ${NODE_IPS[@]};do
        echo ">>> ${node_ip}"
        ssh root@${node_ip} "systemctl status etcd|grep Active"
done
健康检查
for node_ip in ${NODE_IPS[@]};do
        echo ">>> ${node_ip}"
        ETCDCTL_API=3 /usr/bin/etcdctl \
--endpoints=https://${node_ip}:2379 \
--cacert=/etcd/cert/ca.pem \
--cert=/etcd/cert/etcd.pem \
--key=/etcd/cert/etcd-key.pem endpoint health
done 

```

