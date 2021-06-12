# [CentOS7安装使用ab压力测试工具](https://www.cnblogs.com/xingrui/p/10317249.html)

执行安装命令：yum -y install httpd-tools

安装完毕，执行：ab -help,显示命令参数

命令模板：ab -c 100 -n 10000 待测试网站（建议完整路径）

　　-c　　即concurrency，用于指定的并发数

　　-n　　即requests，用于指定压力测试总共的执行次数

