# redis docker-compose部署

#### 编辑配置文件

```yaml
# 创建数据文件夹
mkdir /data

# 创建配置文件夹以及配置文件
mkdir conf
vim conf/sentinel.conf

port 6379
dir "/tmp"
sentinel deny-scripts-reconfig yes
sentinel monitor mymaster 154.8.221.183 16006 1
sentinel failover-timeout mymaster 10000

```

docker-compose文件如下

```yaml
version: '2'
networks:
   iot:
       driver: bridge
services:
  master:
    image: ccr.ccs.tencentyun.com/sfbj/redis:v1.0       ## 镜像
    container_name: redis-master
    command: redis-server --requirepass redis123456
    restart: unless-stopped
    ports:
      - "16006:6379"
    volumes:
      - "./data:/data"
    networks:
      - iot
  slave1:
    image: ccr.ccs.tencentyun.com/sfbj/redis:v1.0                ## 镜像
    container_name: redis-slave-1
    ports:
      - "6380:6379"           ## 暴露端口
    command: redis-server --slaveof redis-master 6379 --requirepass redis123456 --masterauth redis123456
    restart: unless-stopped
    depends_on:
      - master
    volumes:
      - "./data:/data"
    networks:
      - iot
  slave2:
    image: ccr.ccs.tencentyun.com/sfbj/redis:v1.0                ## 镜像
    container_name: redis-slave-2
    ports:
      - "6381:6379"           ## 暴露端口
    command: redis-server --slaveof redis-master 6379 --requirepass redis123456 --masterauth redis123456
    restart: unless-stopped
    depends_on:
      - master
    volumes:
      - "./data:/data"
    networks:
      - iot
  sentinel1:
    image: ccr.ccs.tencentyun.com/sfbj/redis:v1.0       ## 镜像
    container_name: redis-sentinel-1
    privileged: true
    user: root
    ports:
      - "16060:6379"
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    restart: unless-stopped
    depends_on:
      - slave1
    volumes:
      - "./conf/sentinel.conf:/usr/local/etc/redis/sentinel.conf"
    networks:
      - iot
```
