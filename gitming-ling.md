

 Git打标签（Tag）

```
做个笔记记录下来以备自己忘记
列出所有标签
git tag
通配符过滤标签
git tag -l "v1.0.0-RC5*"
新建tag
git tag tagName
创建带备注得tag
git tag -a tagName -m "my tag"
推送tag到远程
git push origin v1.0
删除tag
本地删除：git tag -d v1.0
远程删除：git push origin :refs/tags/v1.0
```



```
1) 切换到基础分支，如主干

git checkout master

2）创建并切换到新分支

git checkout -b panda

git branch可以看到已经在panda分支上

3)更新分支代码并提交

git add *

git commit -m "init panda"

git push origin panda
```

```
使用命令：

git reset --soft HEAD^

这样就成功撤销了commit，如果想要连着add也撤销的话，--soft改为--hard（删除工作空间的改动代码）。

命令详解：

HEAD^ 表示上一个版本，即上一次的commit，也可以写成HEAD~1
如果进行两次的commit，想要都撤回，可以使用HEAD~2

--soft
不删除工作空间的改动代码 ，撤销commit，不撤销git add file

--hard
删除工作空间的改动代码，撤销commit且撤销add
```

![](/assets/import-20210615-0001.png)

