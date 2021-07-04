[\[Centos 7 安装搭建K8S\]](https://www.jianshu.com/p/65ecef9016ae\)

#### 1、环境准备

CentOS Linux release 7.5.1804 \(Core\)  
 内核版本： 3.10.0-862.el7.x86\_64

| 节点 | 主机名 | IP |
| :--- | :--- | :--- |
| Master、etcd、registry | k8s-master | 10.10.10.3 |
| Node1 | k8s-node-1 | 10.10.10.4 |
| Node2 | k8s-node-2 | 10.10.10.5 |

设置主机名：

```
#master主机
[
root@localhost 
~
]
# hostnamectl 
set
-
hostname k8s
-
master

#node1主机
[
root@localhost 
~
]
# hostnamectl 
set
-
hostname k8s
-
node
-
1
#node2主机
[
root@node3 
~
]
# hostnamectl 
set
-
hostname k8s
-
node
-
2
```

关闭系统防火墙及设置selinux状态：

```
[
root@localhost 
~
]
# systemctl stop firewalld 
&
&
 systemctl disable firewalld 
&
&
 setenforce 
0
```

分别配置master及node主机的host文件：

```
[
root@k8s
-
master 
~
]
# vim 
/
etc
/
hosts

10.10
.10
.3
 k8s
-
master

10.10
.10
.3
 etcd

10.10
.10
.3
 registry

10.10
.10
.4
 k8s
-
node
-
1
10.10
.10
.5
 k8s
-
node
-
2
```

在master、node主机上分别配置时间同步：

```
[root@k8s-master ~]# ntpdate ntp1.aliyun.com

```

#### 2、部署master主机

##### 2.1 安装etcd

k8s的运行依赖于etcd，所以需要先部署etcd。

```
[
root@k8s
-
master 
~
]
# yum install 
-
y etcd

```

yum安装的etcd默认的配置文件为/etc/etcd/etcd.conf，编辑配置文件：

```
[
root@k8s
-
master 
~
]
# vim 
/
etc
/
etcd
/
etcd
.
conf 
ETCD_DATA_DIR
=
"/var/lib/etcd/default.etcd"

ETCD_LISTEN_CLIENT_URLS
=
"http://0.0.0.0:2379"

ETCD_NAME
=
"master"

ETCD_ADVERTISE_CLIENT_URLS
=
"http://etcd:2379"
```

启动并验证etcd的运行状态：

```
[
root@k8s
-
master 
~
]
# systemctl start etcd

[
root@k8s
-
master 
~
]
# etcdctl 
-
C
 http
:
/
/
etcd
:
2379
 cluster
-
health
member 
8
e9e05c52164694d 
is
 healthy
:
 got healthy result 
from
 http
:
/
/
etcd
:
2379

cluster 
is
 healthy

```

##### 2.2 安装Docker和Docker私有仓库

```
[
root@k8s
-
master 
~
]
# yum install docker  docker
-
distribution 
-
y 

```

配置Docker 配置文件，允许从registry中拉取镜像。

```
[root@k8s-master ~]# vim /etc/sysconfig/docker
OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi

OPTIONS='--insecure-registry registry:5000'

```

设置开机自启动并开启服务：

```
[
root@k8s
-
master 
~
]
# systemctl start docker docker
-
distribution

[
root@k8s
-
master 
~
]
# systemctl enable docker docker
-
distribution

```

##### 2.3 安装kubernetes

```
[
root@k8s
-
master 
~
]
# yum install 
-
y kubernetes

```

在master 主机上需要运行的kubernetes 组件有：kubernetes API server，kubernetes Controller Manager ，Kubernetes Scheduler，需要分别修改下述对应配置：

###### 2.3.1 /etc/kubernetes/apiserver

```
[
root@k8s
-
master 
~
]
# vim 
/
etc
/
kubernetes
/
apiserver
KUBE_API_ADDRESS
=
"--insecure-bind-address=0.0.0.0"

KUBE_ETCD_SERVERS
=
"--etcd-servers=http://etcd:2379"

KUBE_SERVICE_ADDRESSES
=
"--service-cluster-ip-range=10.254.0.0/16"

KUBE_ADMISSION_CONTROL
=
"--admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"

KUBE_API_ARGS
=
""
```

###### 2.3.2 /etc/kubernetes/config

```
[
root@k8s
-
master 
~
]
# grep 
-
v 
"^#"
/
etc
/
kubernetes
/
config
KUBE_LOGTOSTDERR
=
"--logtostderr=true"

KUBE_LOG_LEVEL
=
"--v=0"

KUBE_ALLOW_PRIV
=
"--allow-privileged=false"

KUBE_MASTER
=
"--master=http://k8s-master:8080"
```

最后分别启动并设置开机自启动：

```
[
root@k8s
-
master 
~
]
# systemctl enable kube
-
apiserver kube
-
controller
-
manager kube
-
scheduler

[
root@k8s
-
master 
~
]
# systemctl start kube
-
apiserver kube
-
controller
-
manager kube
-
scheduler

```

#### 3、部署node主机

##### 3.1 安装docker

参考2.2安装docker，不需要安装docker-distribution。

##### 3.2 安装kubernetes

```
[
root@k8s
-
node
-
1
~
]
# yum install 
-
y kubernetes

```

在node节点上需要启动kubernetes 下述组件：kubelet、kubernets-Proxy，因此需要相应修改下述配置。

###### 3.2.1 /etc/kubernetes/config

```
[
root@k8s
-
node
-
1
~
]
# vim 
/
etc
/
kubernetes
/
config 
KUBE_LOGTOSTDERR
=
"--logtostderr=true"

KUBE_LOG_LEVEL
=
"--v=0"

KUBE_ALLOW_PRIV
=
"--allow-privileged=false"

KUBE_MASTER
=
"--master=http://k8s-master:8080"
```

###### 3.2.2 /etc/kubernetes/kubelet

```
[
root@k8s
-
node
-
1
~
]
# vim 
/
etc
/
kubernetes
/
kubelet
KUBELET_ADDRESS
=
"--address=0.0.0.0"

KUBELET_HOSTNAME
=
"--hostname-override=k8s-node-1"
#注意配置为对应node的hostname

KUBELET_API_SERVER
=
"--api-servers=http://k8s-master:8080"

KUBELET_POD_INFRA_CONTAINER
=
"--pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest"

KUBELET_ARGS
=
""
```

最后设置开机自启动：

```
[
root@k8s
-
node
-
1
~
]
# systemctl enable kubelet kube
-
proxy

[
root@k8s
-
node
-
1
~
]
# systemctl start kubelet kube
-
proxy

```

在master 上查看集群节点状态：

```
[
root@k8s
-
master 
~
]
# kubectl 
-
s http
:
/
/
k8s
-
master
:
8080
 get node
NAME STATUS AGE
k8s
-
node
-
1
 Ready 
6
m
k8s
-
node
-
2
 Ready 
13
s

```

#### 4、安装覆盖网络-Flannel

##### 4.1 yum 安装 Flannel

```
[root@k8s-master ~]# yum install -y flannel
....

Installed
:

  flannel.x86_64 0
:
0.7.1-4.el7 

```

##### 4.2 配置Flannel

分别在master和node 主机上配置/etc/sysconfig/flanneld配置文件，如：

```
[
root@k8s
-
master 
~
]
# vim 
/
etc
/
sysconfig
/
flanneld
FLANNEL_ETCD_ENDPOINTS
=
"http://etcd:2379"

FLANNEL_ETCD_PREFIX
=
"/atomic.io
etwork"
```

##### 4.3 配置etcd中关于flannel的key

由于Flannel 是使用Etcd来进行配置保证多个Flannel实例之间的配置的一致性，因此需要在etcd上进行网段key的配置（‘/atomic.io/network/config’这个key与上文/etc/sysconfig/flanneld中的配置项FLANNEL\_ETCD\_PREFIX是相对应的，错误的话启动就会出错\)

```
[
root
@
k8s
-
master 
~
]
# etcdctl mk 
/
atomic
.
io
/
network
/
config 
'{"Network": "10.88.0.0/16"}'
{
"Network"
:
"10.88.0.0/16"
}
```

最后启动Flannel 并依次重启docker和kubernetes。

```
#在master 执行
[
root
@k8s
-
master 
~
]
# systemctl enable flanneld
[
root
@k8s
-
master 
~
]
# systemctl start flanneld
[
root
@k8s
-
master 
~
]
# systemctl restart docker kube-apiserver kube-controller-manager kube-scheduler
#在node上执行
[
root
@k8s
-
node
-
1
~
]
# systemctl enable flanneld
[
root
@k8s
-
node
-
1
~
]
# systemctl start flanneld

root
@k8s
-
node
-
1
~
]
# systemctl restart kubelet kube-proxy
```

部署完成后需要安装podinfrastructure 才能部署pod，相应的安装方式可参考下述连接  
 参考：[https://www.cnblogs.com/zhenyuyaodidiao/p/6500897.html](https://www.cnblogs.com/zhenyuyaodidiao/p/6500897.html)  
[https://blog.csdn.net/fei79534672/article/details/78710858](https://blog.csdn.net/fei79534672/article/details/78710858)  
[https://www.cnblogs.com/zhenyuyaodidiao/p/6500830.html](https://www.cnblogs.com/zhenyuyaodidiao/p/6500830.html)

