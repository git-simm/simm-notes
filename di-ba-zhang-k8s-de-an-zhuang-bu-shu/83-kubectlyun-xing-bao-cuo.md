# The connection to the server localhost:8080 was refused - did you specify the right host or port? {#articleContentId}

解决k8s集群在节点运行kubectl出现的错误：The connection to the server localhost:8080 was refused - did you specify the right host or port?

问题分析：

出现这个问题的原因是kubectl命令需要使用kubernetes-admin来运行

1

解决办法：

将主节点（master节点）中的【/etc/kubernetes/admin.conf】文件拷贝到从节点相同目录下:

scp -r /etc/kubernetes/admin.conf ${node1}:/etc/kubernetes/admin.conf

配置环境变量：

echo “export KUBECONFIG=/etc/kubernetes/admin.conf” &gt;&gt; ~/.bash\_profile

立即生效：

source ~/.bash\_profile

具体操作截图：

![](/assets/import23.png)

```
Process: 1532 ExecStart=/usr/bin/kube-apiserver $KUBE_LOGTOSTDERR $KUBE_LOG_LEVEL $KUBE_ETCD_SERVERS $KUBE_API_ADDRESS $KUBE_API_PORT $KUBELET_PORT $KUBE_ALLOW_PRIV $KUBE_SERVICE_ADDRESSES $KUBE_ADMISSION_CONTROL $KUBE_API_ARGS (code=exited, status=2)
 Main PID: 1532 (code=exited, status=2)
 
```



