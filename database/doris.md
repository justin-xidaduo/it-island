---
description: docker-compose安装部署
---

# doris

#### 打包以及编译

```
wget https://github.com/apache/incubator-doris/archive/refs/tags/0.14.0.tar.gz
tar -zxvf 0.14.0.tar.gz
mv incubator-doris-0.14.0 0.14.0
mkdir source
mv 0.14.0 source

#####################

# 编写build.yaml

version: "3.7"

networks:
  doris-build:
    name: doris-build
    driver: bridge

services:
  doris-build:
    image: apachedoris/doris-dev:build-env-1.2
    container_name: dev
    hostname: dev
    privileged: true
    command: ["source/0.14.0/build.sh"]
    volumes:
      - ./data/.m2:/root/.m2
      - ./source:/root/source
    networks:
      - doris-build

docker-compose up -f build.yaml

#########################
编译过程比较漫长，耐心等待
```

#### 制作镜像

```
mv source/0.14.0/output .
mv source/0.14.0/fs_brokers/apache_hdfs_broker
```

#### Dockerfile 编写

```
FROM centos:7.5.1804

RUN \
	yum clean metadata && \
	yum -y install epel-release && \
	yum clean metadata && \
	yum -y install vim  make which tar rpm-build yum-utils python-argparse python-yaml java-1.8.0-openjdk-devel gcc gcc-c|| createrepo jq glibc-static krb5-workstation openssh-clients && \
	yum clean all

ENV JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk

RUN mkdir -p /home/doris

COPY ./output/fe /home/doris/fe

COPY ./output/be /home/doris/be

COPY ./output/apache_hdfs_broker /home/doris/fs_broker

```

```
docker build --pull --rm -f Dockerfile -t doris:0.14.00 .
```

#### 创建目录，拷贝配置文件

```
mkdir ./doris/be/conf/
mkdir ./doris/fe/fe-follower-1/conf/
mkdir ./doris/fe/fe-observer-1/conf/
mkdir ./doris/fe/leader/conf/
 
cp be.conf ./doris/be/conf/be.conf
cp fe.conf ./doris/fe/fe-follower-1/conf/
cp fe.conf ./doris/fe/fe-observer-1/conf/
cp fe.conf ./doris/fe/leader/conf/
```

#### 文件夹tree

![](<../.gitbook/assets/image (16).png>)

#### docker-compose.yaml

```
version: "3.7"
networks:
  doris:
    name: doris
    driver: bridge
 
services:
  fe-leader:
    image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/doris:0.14.00
    restart: always
    container_name: fe-leader
    hostname: fe-leader
    environment:
      - TZ=Asia/Shanghai
    command: /home/doris/fe/bin/start_fe.sh
    volumes:
      - ./doris/fe/leader/log:/home/doris/fe/log
      - ./doris/fe/leader/doris-meta:/home/doris/fe/doris-meta
      - ./doris/fe/leader/conf:/home/doris/fe/conf
      - /etc/localtime:/etc/localtime:ro
    stdin_open: true
    tty: true
    ports:
      - 8030:8030
      - 8040:8040
      - 9030:9030
    networks:
      - doris
 
  fe-follower-1:
    image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/doris:0.14.00
    restart: always
    container_name: fe-follower-1
    hostname: fe-follower-1
    environment:
      - TZ=Asia/Shanghai
    command: /home/doris/fe/bin/start_fe.sh --helper fe-leader:9010
    volumes:
      - ./doris/fe/fe-follower-1/log:/home/doris/fe/log
      - ./doris/fe/fe-follower-1/doris-meta:/home/doris/fe/doris-meta
      - ./doris/fe/fe-follower-1/conf:/home/doris/fe/conf
      - /etc/localtime:/etc/localtime:ro
    stdin_open: true
    tty: true
    security_opt:
      - seccomp:unconfined
    privileged: true
    depends_on:
      - fe-leader
    networks:
      - doris
 
  fe-observer-1:
    image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/doris:0.14.00
    restart: always
    container_name: fe-observer-1
    hostname: fe-observer-1
    environment:
      - TZ=Asia/Shanghai
    command: /home/doris/fe/bin/start_fe.sh --helper fe-leader:9010
    volumes:
      - ./doris/fe/fe-observer-1/log:/home/doris/fe/log
      - ./doris/fe/fe-observer-1/doris-meta:/home/doris/fe/doris-meta
      - ./doris/fe/fe-observer-1/conf:/home/doris/fe/conf
 
      - /etc/localtime:/etc/localtime:ro
    stdin_open: true
    tty: true
    security_opt:
      - seccomp:unconfined
    privileged: true
    depends_on:
      - fe-leader
    networks:
      - doris
 
  be:
    image: registry.cn-huhehaote.aliyuncs.com/zlb_dockerhub/doris:0.14.00
    restart: always
    container_name: be
    hostname: be
    environment:
      - TZ=Asia/Shanghai
    command: /home/doris/be/bin/start_be.sh
    volumes:
      - "./doris/be/log:/home/doris/be/log"
      - "./doris/be/storage:/home/doris/be/storage"
      - "./doris/be/conf:/home/doris/be/conf/"
      - "/etc/localtime:/etc/localtime:ro"
    stdin_open: true
    tty: true
    security_opt:
      - seccomp:unconfined
    depends_on:
      - fe-leader
    networks:
      - doris
 
  mysql:
    image: "mysql:5.7.23"
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_USER: "root"
      MYSQL_PASS: "123456"
      TZ: Asia/Shanghai
    volumes:
      - ./doris/mysql:/var/lib/mysql
 #     - ./doris/mysql:/docker-entrypoint-initdb.d/
 #     - ./doris/mysql/sources.list:/etc/apt/sources.list
    command: [
            "--log-bin=mysql-bin",
            "--server-id=1",
            "--character-set-server=utf8mb4",
            "--collation-server=utf8mb4_unicode_ci",
            "--innodb_flush_log_at_trx_commit=1",
            "--sync_binlog=1"
        ]
    depends_on:
      - fe-leader
    networks:
      - doris
```

```
docker-compose up -d
```

#### 配置及添加

```
# 进入mysql容器
docker exec -it mysql /bin/bash
 
#添加 follower
mysql -h fe-leader -u root -P 9030 <<EOF
    ALTER SYSTEM ADD FOLLOWER "fe-follower-1:9010";
EOF
 
#添加 observer
mysql -h fe-leader -u root -P 9030 <<EOF
    ALTER SYSTEM ADD OBSERVER "fe-observer-1:9010";
EOF
 
 
#添加be
mysql -h fe-leader -u root -P 9030 <<EOF
    ALTER SYSTEM ADD BACKEND "be:9050";
EOF
 
 
#查看各容器状态
mysql -h fe-leader -u root -P 9030 <<EOF
    SHOW PROC '/frontends';
EOF
```
