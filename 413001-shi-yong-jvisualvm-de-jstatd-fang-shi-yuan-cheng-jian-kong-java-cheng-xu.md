# [利用VisualVm远程监控Java进程](https://segmentfault.com/a/1190000016634627)

[    
](https://segmentfault.com/u/chanjarster)转载：[https://segmentfault.com/a/1190000016634627](https://chanjarster.github.io/post/visualvm-remote-monitoring/)

本文介绍利用[VisualVm](https://visualvm.github.io/)和[jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)来远程监控Java进程的方法。

要实现远程监控Java进程，必须在远程主机（运行Java程序的主机）上跑一个[jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)进程，这个进程相当于一个agent，用来收集远程主机上的JVM运行情况，然后用[VisualVm](https://visualvm.github.io/)连接到这个[jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)，从而实现远程监控的目的。

## 第一步：在远程主机上启动jstatd {#item-1}

要注意的是，[jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)是一个RMI server application，因此在启动时支持[java.rmi properties](https://docs.oracle.com/javase/8/docs/technotes/guides/rmi/javarmiproperties.html)。

根据[jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)文档，我们需要在启动[jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)时提供一个security policy文件：

```
grant codebase 
"file:${java.home}/../lib/tools.jar"
 {   
    permission java.security.AllPermission
;

}
;
```

然后运行下面命令启动：

```
jstatd -J-Djava.security.policy=jstatd.all.policy
```

**不过这里有一个陷阱**，见SO上的这个提问：[VisualVm connect to remote jstatd not showing applications](https://stackoverflow.com/a/33219226/1287790)。在启动时还得指定rmi server hostname，否则VisualVm无法看到远程主机上的Java进程。所以正确的命令应该是这样：

```
jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=
<
host or ip
>
```

远程主机的hostname可以随便填，只要VisualVm能够ping通这个hostname就行了。所以说下面这几种情况都是可行的：

* 远程主机没有DNS name，但VisualVm所在主机的
  `/etc/hosts`
  里配置了
  `some-name`
  `<`
  `ip-to-remote-host`
  `>`
  。jstatd启动时指定
  `-J-Djava.rmi.server.hostname=some-name`
  ，VisualVm连接
  `some-name`
  。
* 远程主机经过层层NAT，它的内部ip比如是
  `192.168.xxx.xxx`
  ，它的对外的NAT地址是
  `172.100.xxx.xxx`
  。jstatd启动时指定
  `-J-Djava.rmi.server.hostname=172.100.xxx.xxx`
  ，VisualVm连接
  `172.100.xxx.xxx`
  。
* 上面两种方式混合，即在VisualVm所在主机的
  `/etc/hosts`
  里配置
  `some-name`
  `<`
  `ip-to-remote-host-nat-address`
  `>`
  。jstatd启动时指定
  `-J-Djava.rmi.server.hostname=some-name`
  ，VisualVm连接
  `some-name`
  。

还有要注意一点，运行`jstatd`的用户必须和运行Java程序的用户相同，或者是root，否则会监控不到远程主机上的Java进程。

## 第二步：启动VisualVm {#item-2}

在你的机器上运行`jvisualvm`启动VisualVm。按照下面步骤添加远程主机：

**第一步**

![](https://segmentfault.com/img/bVbhXAw?w=458&h=212 "图片描述")

**第二步**

![](https://segmentfault.com/img/bVbhXAx?w=760&h=334 "图片描述")

**第三步**

![](https://segmentfault.com/img/bVbhXAy?w=422&h=246 "图片描述")

你就能看到远程主机上的Java进程了。

需要注意的是如果你点开一个远程进程，那么你会发现有些信息是没有的，比如：CPU、线程、和MBeans。这是正常的，如果需要这些信息（就像监控本地Java进程一样），那么就需要用JMX，相关内容会在另一篇文章中讲解。

## 参考资料 {#item-3}

* [VisualVm - Working with Remote Applications](https://htmlpreview.github.io/?https://raw.githubusercontent.com/visualvm/visualvm.java.net.backup/master/www/applications_remote.html)
* [jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)
* [java.rmi Properties](https://docs.oracle.com/javase/8/docs/technotes/guides/rmi/javarmiproperties.html)
* [VisualVm connect to remote jstatd not showing applications](https://stackoverflow.com/a/33219226/1287790)



