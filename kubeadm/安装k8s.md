## 安装

```
配置仓库(所有节点)
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

```
安装kubernetes相关组件（所有节点）

yum install -y kubelet-1.15.0-0.x86_64  kubectl-1.15.0-0.x86_64  kubeadm-1.15.0-0.x86_64

systemctl enable kubelet && systemctl start kubelet
```

```
下载镜像（所有节点）
使用脚本
#!/bin/bash
export tag=v1.15.0

images=(kube-apiserver:$tag kube-controller-manager:$tag kube-scheduler:$tag etcd:3.3.10 kube-proxy:$tag pause:3.1 )

for image in ${images[@]}; do 
docker pull mirrorgooglecontainers/$image 
docker tag  mirrorgooglecontainers/$image k8s.gcr.io/$image
docker rmi mirrorgooglecontainers/$image
done

docker pull coredns/coredns:1.3.1
docker tag coredns/coredns:1.3.1  k8s.gcr.io/coredns:1.3.1
docker rmi coredns/coredns:1.3.1

执行脚本

```

```
初始化集群（master）
kubeadm init --kubernetes-version=v1.15.0
稍等一会会出现
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.100.94:6443 --token n4j6b0.qeq0w4etzdzjmihw \
    --discovery-token-ca-cert-hash sha256:75166120f939fd625575f8b081337a05bf71e72ed0f563f1b7442a9f9400c5c9 
    
 
在要加入的节点执行（node）
kubeadm join 192.168.100.94:6443 --token n4j6b0.qeq0w4etzdzjmihw \
    --discovery-token-ca-cert-hash sha256:75166120f939fd625575f8b081337a05bf71e72ed0f563f1b7442a9f9400c5c9
    
配置kubectl(管理节点)
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

```
验证集群
kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"} 
```

```
安装flannel

kubectl apply -f flannel.yaml
```

