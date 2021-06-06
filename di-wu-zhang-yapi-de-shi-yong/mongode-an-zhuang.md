[https://zhuanlan.zhihu.com/p/148054551](https://zhuanlan.zhihu.com/p/148054551)

# docker安装MongoDB数据库

## 在dockerHub中找到MongoDB

[dockerHub](https://link.zhihu.com/?target=https%3A//hub.docker.com/_/mongo)

## 使用docker-compose来安装

```
version: '3.1'
services:
  mongo:
    image: mongo
    restart: always
    environment:
      #用户名密码
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
    ports:
      - 27010:27017
    volumes: 
      - /home/mongo-data:/data/db
```

## 使用docker-compose来执行yml文件

```
docker-compose up -d
```

## 连接mongo

> 在docker中安装的mongo,使用docker的交互式终端连接

```
# 进入
docker exec -it mongo-data_mongo_1 mongo
# 进入
MongoDB shell version v4.2.7
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled
&
gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("7749cab4-9541-4aa6-a070-983b687992b0") }
MongoDB server version: 4.2.7
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
    http://docs.mongodb.org/
Questions? Try the support group
    http://groups.google.com/group/mongodb-user

>
```

> 注意, 这里只是进入了mongo的交互终端,还需要登录设置密码

## 登录

```
>
 show dbs # 没有登录

>
 use admin
switched to db admin

>
 db.auth('root','example') # 输入刚刚yml中写入的用户名和密码
1 # 返回1就是ok了

>
 show dbs # 可以看到数据库了
admin   0.000GB
config  0.000GB
local   0.000GB

>
```

## 创建数据库

```
use testdb
```

## 创建一个用户

```
db.createUser({
    user:'test',
    pwd:'123456',
    roles:[
        {
            role:'dbOwner',
            db:'testdb'
        }
    ]
})
# 返回信息,成功创建一个新用户
Successfully added user: {
    "user" : "test",
    "roles" : [
        {
            "role" : "dbOwner",
            "db" : "testdb"
        }
    ]
}
```

## 使用新用户登录

```
>
 use testdb
switched to db testdb

>
 db.auth('test','123456')
1

>
```

## 来插入一条数据

```
>
 db.users.insertOne({name:'junjun',age:22,emali:'11776174@qq.com'})
{
    "acknowledged" : true,
    "insertedId" : ObjectId("5ee1e6b45b1e5ccda822af67")
}
```

## 显示数据库

```
>
 show collections
users
```

## 查询数据库

```
>
 db.users.find({})
{ "_id" : ObjectId("5ee1e6b45b1e5ccda822af67"), "name" : "junjun", "age" : 22, "email" : "11776174@qq.com" }
```

## 修改数据库

```
# 修改

>
 db.users.updateOne({name:'junjun2'},{$set:{email:'11776174@qq.com'}})
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
# 查询

>
 db.users.find({})
{ "_id" : ObjectId("5ee1e6b45b1e5ccda822af67"), "name" : "junjun", "age" : 22, "email" : "11776174@qq.com" }
{ "_id" : ObjectId("5ee1e7e05b1e5ccda822af68"), "name" : "junjun2", "age" : 10, "email" : "11776174@qq.com" }

>
```

## 删除数据库

```
>
 db.users.deleteOne({name:"junjun2"})
{ "acknowledged" : true, "deletedCount" : 1 }

>
 db.users.find({})
{ "_id" : ObjectId("5ee1e6b45b1e5ccda822af67"), "name" : "junjun", "age" : 22, "email" : "11776174@qq.com" }

>
```

## mongodb的备份与恢复

### 备份

```
docker exec -it mongo-data_mongo_1 mongodump -h localhost -u root -p example -o /tmp/test
# -h 地址
# -u 用户名
# -p 密码
# -o 备份的路径
2020-06-11T09:10:33.142+0000    writing admin.system.users to 
2020-06-11T09:10:33.142+0000    done dumping admin.system.users (2 documents)
2020-06-11T09:10:33.143+0000    writing admin.system.version to 
2020-06-11T09:10:33.143+0000    done dumping admin.system.version (2 documents)
2020-06-11T09:10:33.143+0000    writing testdb.users to 
2020-06-11T09:10:33.144+0000    done dumping testdb.users (1 document)
```

### 把容器内部的数据拷贝出来

```
docker cp mongo-data_mongo_1:/tmp/test /tmp/test
```

### 恢复

```
docker exec -it mongo-data_mongo_1 mongorestore -h localhost -u root -p example --dir /tmp/test
# -h 地址
# -u 用户名
# -p 密码
# --dir  备份的路径
```



