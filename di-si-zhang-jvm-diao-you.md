# VisualVM使用jstatd远程监控jvm

## 配置安全策略

1. 在jdk的bin目录下创建文件jstatd.all.policy
2. 写入下面的安全配置

```
grant codebase "file:/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.191.b12-1.el7_6.x86_64/lib/tools.jar" {
   permission java.security.AllPermission;
};

```

此处写绝对路径，主要是防止路径错误问题，排查问题，应该写成相对路径。

## 启动jstatd

```
./jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.hostname=106.15.95.37 
&
```

## 查看jstatd是否启动成功

```
[
root
@sunpyServer
 bin
]
# jps -l       
3040
 sun
.
tools
.
jstatd
.
Jstatd
3094
 sun
.
tools
.
jps
.
Jps
[
root
@sunpyServer
 bin
]
# netstat -anp|grep jstatd

tcp        
0
0
0.0
.0
.0
:
1099
0.0
.0
.0
:
*
LISTEN
3040
/
.
/
jstatd       
tcp        
0
0
0.0
.0
.0
:
38090
0.0
.0
.0
:
*
LISTEN
3040
/
.
/
jstatd       
tcp        
0
1
172.19
.120
.40
:
54572
106.15
.95
.37
:
38090
SYN_SENT
3040
/
.
/
jstatd       
tcp        
0
0
172.19
.120
.40
:
1099
172.19
.120
.40
:
34176
ESTABLISHED
3040
/
.
/
jstatd       
tcp        
0
0
172.19
.120
.40
:
34176
172.19
.120
.40
:
1099
ESTABLISHED
3040
/
.
/
jstatd       
unix  
2
[
]
STREAM
CONNECTED
31805
3040
/
.
/
jstatd 

```

## 开放安全组

踩坑：

```
C
:
\Users\Administrator
>
jps 106.15.95.37
RMI Server JStatRemoteHost not available

```

原因：  
 我使用的阿里云服务器，对外开放安全组端口为1099，但是实际上本地jps 106.15.95.37根本没用。因为1099是内网地址对应的端口。而外网访问的端口是38090。所以我开放38090端口后就可以访问了。

```
C
:
\Users\Administrator
>
jps 106.15.95.37
3040 Jstatd

```

![](/assets/import23324.png)



