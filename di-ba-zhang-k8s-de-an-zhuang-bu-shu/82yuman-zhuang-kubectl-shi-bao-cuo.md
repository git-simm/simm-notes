# [yum安装kubectl时报错](https://www.cnblogs.com/peteremperor/p/12493677.html)

报错信息：file /usr/bin/kubectl from install of kubectl-1.17.4-0.x86\_64 confliccts with file from package kubernetes-clients-1.5.2-0.7.git269f928.el7.x86\_64

安装过其他版本或包，卸载掉再装就好了

```
yum remove kubernetes-client-1.5.2-0.7.git269f928.el7.x86_64
```

再重新安装：

```
yum install -y kubeadm
```

Installed:  
kubeadm.x86\_64 0:1.14.2-0

Dependency Installed:  
cri-tools.x86\_64 0:1.12.0-0 kubectl.x86\_64 0:1.14.2-0 kubelet.x86\_64 0:1.14.2-0 kubernetes-cni.x86\_64 0:0.7.5-0

Complete!

