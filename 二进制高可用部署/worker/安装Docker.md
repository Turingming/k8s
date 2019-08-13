## 安装Docker

```
配置依赖及仓库

yum install yum-utils device-mapper-persistent-data lvm2 -y 

yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo


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

mkdir -p /etc/systemd/system/docker.service.d

```



```
启动并验证

systemctl daemon-reload && systemctl enable docker && systemctl start docker && systemctl status  docker

```

