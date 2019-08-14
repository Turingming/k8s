## 安装Docker

```
配置依赖及仓库
yum-config-manager \
  --add-repo \
  https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
  
yum update && yum install docker-ce-18.06.2.ce -y 
```

```
由于cgroup由两种方式实现，需要更改成systemd

mkdir /etc/docker

cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

```



```
启动并验证

systemctl daemon-reload && systemctl enable docker && systemctl start docker && systemctl status  docker

```

