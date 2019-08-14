```
二进制安装下载地址：
    https://github.com/helm/helm/releases
    把helm移动到/usr/bin目录下
```

```
执行helm init

kubectl edit deployment  tiller-deploy  -n kube-system  #将tiller镜像从国外镜像换成国内镜像

yum install -y socat
```

```
授权
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system	
    
    
  kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'  

```

