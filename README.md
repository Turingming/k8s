# kubernetes 1.15.0 集群部署
1.二进制安装高可用kubernetes 

2.addons

- coredns       #部署dns

- storagec

- ```
  storageclass.yaml   #创建glusterfs驱动，通过storageclass动态分配空间
  heketi-secret.yaml  #创建glusterfs认证
  pvc.yaml            #测试
  ```

  

