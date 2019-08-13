## Master高可用搭建

```
keepalived 提供 kube-apiserver 对外服务的 VIP；
```



```
haproxy 监听 VIP，后端连接所有 kube-apiserver 实例，提供健康检查和负载均衡功能；haproxy 监听的端口(8443) 需要与 kube-apiserver 的端口 6443 不同，避免冲突
```

```
所有组件都通过 VIP 和 haproxy 监听的 8443 端口访问 kube-apiserver 服务
```

```
安装
yum install -y keepalived haproxy
```

```
配置keepalived
cat /etc/keepalived/keepalived.conf
global_defs {
    router_id apiserver
}

vrrp_script check-haproxy {
    script "killall -0 haproxy"
    interval 3
}

vrrp_instance VI-kube-master {
    state BACKUP
    priority 120           （另外两节点110，100）
    dont_track_primary     #设置不抢占，必须设置在backup上且priority最高的节点上
    interface ens37        #网卡
    virtual_router_id 68
    advert_int 3
    track_script {
        check-haproxy
    }
    virtual_ipaddress {
        192.168.100.252　　　　#VIP，访问此IP调用api-server
    }
}



启动服务
systemctl enable keepalived && systemctl start keepalived
```

```
配置haproxy
/etc/haproxy/haproxy.cfg

global
    log /dev/log    local0
    log /dev/log    local1 notice
    chroot /var/lib/haproxy
    stats socket /var/run/haproxy-admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon
    nbproc 1

defaults
    log     global
    timeout connect 5000
    timeout client  10m
    timeout server  10m

listen  admin_stats
    bind 0.0.0.0:10080
    mode http
    log 127.0.0.1 local0 err
    stats refresh 30s
    stats uri /status
    stats realm welcome login\ Haproxy
    stats auth admin:123456
    stats hide-version
    stats admin if TRUE

listen kubernetesmaster
    bind 0.0.0.0:8443    #监听
    mode tcp
    option tcplog
    balance roundrobin
    server 192.168.100.245 192.168.100.245:6443 check inter 2000 fall 2 rise 2 weight 1
    server 192.168.100.246 192.168.100.246:6443 check inter 2000 fall 2 rise 2 weight 1
    server 192.168.100.247 192.168.100.247:6443 check inter 2000 fall 2 rise 2 weight 1



```

