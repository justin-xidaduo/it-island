# 基本操作备忘

## yum 安装mongo

```
# 配置yum源
vim /etc/yum.repos.d/mongo.repo
[mngodb-org]
name=MongoDB Repository
baseurl=http://mirrors.aliyun.com/mongodb/yum/redhat/7Server/mongodb-org/4.0/x86_64/
gpgcheck=0
enabled=1

yum makecache

## 安装mongo
yum install mongodb-org -y

## 启动及开机自启
systemctl start mongo
systemctl restart mongo
systemctl enable mongo
```

## 数据导出导入

```
#导出
mongodump -u mongouser -p mongouser1234 --host *.*.*.*  --port 9280 --authenticationDatabase admin -d app_version -o /backup/app_version_2020_05_27.bak

#导入
mongorestore -d app_version --dir ./app_version
```

## 其它

```
#### 基本使用
#连接mongo数据库
mongo  --host 127.0.0.1 --port 9280 --username mongouser --password mongouser1234 --authenticationDatabase=admin


mongo 172.16.10.14:9280/authority -u mongouser -p mongouser1234 --authenticationDatabase=admin


#查看所有的库
show dbs

#查看表
show tables

#备份单个库（app_version)
mongodump -u mongouser -p mongouser1234  --port 9280 --authenticationDatabase admin -d app_version -o /backup/app_version_2020_04_23.bak

#设置用户名密码
db.createUser( { user:"mongouser", pwd:"mongouser1234", roles:[{role:"root",db:"admin"}] } );


db.createUser({
    user:"mongouser",
    pwd:"mongouser1234",
roles:[
    {role:"dbOwner",db:"sfy-map-data"},
     ]
 })
 
 
 ## 删除索引
 
 1、map-element 的index 删除：
### 查看map-element 的索引：
db.map_element.getIndexes()
###删除
db.map_element.dropIndex('elementId_1')

## 权限相关
- 常用的role值记录: 
    1. 数据库用户角色：read、readWrite; 
    2. 数据库管理角色：dbAdmin、dbOwner、userAdmin； 
    3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager； 
    4. 备份恢复角色：backup、restore； 
    5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase 
    6. 超级用户角色：root 
    7. 内部角色：__system 
- 相应的功能 - 
    Read：允许用户读取指定数据库 
    - readWrite：允许用户读写指定数据库 
    - dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile 
    - userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户 
    - clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。 
    - readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限 
    - readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限 
    - userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限 
    - dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。 
    - root：只在admin数据库中可用。超级账号，超级权限
    
    
## 设置权限

# 查看所有用户
use admin
db.system.users.find().pretty()


# 创建超级管理员
db.createUser({user:"root", pwd:"123456",roles:[{role:"root",db:"admin"}]})


```

