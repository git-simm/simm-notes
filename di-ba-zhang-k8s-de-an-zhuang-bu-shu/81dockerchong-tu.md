# [error:docker-ce conflicts with 2:docker-1.13.1-74.git6e3bb8e.el7.centos.x86\_64](https://www.cnblogs.com/user-sunli/p/13848453.html)

问题原因：安装docker之前有安装cockpit-docker服务

解决方法：卸载docker-ce

```
[root@localhost ~]# yum list installed |
 grep docker
docker
-ce.x86_64 
18.06
.
1
.ce-
3
.el7 @docker-ce-
stable

[root@localhost 
~]# yum remove -y docker-
ce.x86_64

[root@localhost 
~]# mv /
var
/lib/docker /tmp/
```



