---
description: docker
---

# 🍋 Docker

## 1、原理

### 1.1、容器是如何进行隔离的？

在创建新进程时，通过 Namespace 技术，如 PID namespaces 等，实现隔离性。让运行后的容器仅能看到本身的内容。比如，在容器运行时，会默认加上 PID, UTS, network, user, mount, IPC, cgroup 等 Namespace.

### 1.2、容器是如何进行资源限制的？

通过 Linux Cgroup 技术，可为每个进程设定限制的 CPU，Memory 等资源，进而设置进程访问资源的上限。

### 1.3、简述下 docker 的文件系统？

docker 的文件系统称为 rootfs，它的实现的想法来自与 Linux unionFS 。将不同的目录，挂载到一起，形成一个独立的视图。并且 docker 在此基础上引入了层的概念，解决了可重用性的问题。

在具体实现上，rootfs 的存储区分根据 linux 内核和 docker 本身的版本，分为 `overlay2` , `overlay`, `aufs`, `devicemapper` 等。rootfs（镜像）其实就是多个层的叠加，当多层存在相同的文件时，上层的文件会将下层的文件覆盖掉。

### 1.4、容器的启动过程？

1. 指定 Linux Namespace 配置
2. 设置指定的 Cgroups 参数
3. 切换进程的根目录

### 1.5、容器内运行多个应用的问题？

首先更正一个概念，我们都说容器是一个单进程的应用，其实这里的单进程不是指在容器中只允许着一个进程，而是指只有一个进程时可控的。在容器内当然可以使用 ping，ssh 等进程，但这些进程时不受 docker 控制的。

容器内的主进程，也就是 pid =1 的进程，一般是通过 DockerFile 中 ENTRYPOINT 或者 CMD 指定的。如果在一个容器内如果存在着多个服务（进程），就可能出现主进程正常运行，但是子进程退出挂掉的问题，而对于 docker 来说，仅仅控制主进程，无法对这种意外的情况作出处理，也就会出现，容器明明正常运行，但是服务已经挂掉的情况，这时编排系统就变得非常困难。而且多个服务，在也不容易进行排障和管理。

所以如果真的想要在容器内运行多个服务，一般会通过带有 `systemd` 或者 `supervisord` 这类工具进行管理，或者通过 `--init` 方法。其实这些方法的本质就是让多个服务的进程拥有同一个父进程。

## 2、dockerfile

### FROM

尽可能的使用官方仓库存储的镜像作为基础镜像。官方建议使用 Alpine，它大小仅5mb左右，麻雀虽小五脏俱全，用户态工具基本都有。

**建议：私有registry存在时，可以通过官方镜像from下来自己维护，可以自定义调整时区、基础命令等**

### LABEL

我们可以给镜像添加标签（LABEL），如记录仓库地址，维护人联系方式等等。label标签以键值对的形式出现，如果包含空格请用""扩起来。标签对象必须唯一，否则后者会覆盖前者。键可以包含 .、-、a-zA-Z、0-9。下面是一些例子：

```shell
# Set one or more individual labels
LABEL maintainer="Yichen Wang <wangycc1028@gmail.com>" \
       reference="https://github.com/wangycc/jumpserver" \
       nodejs=5.12.0 \
       tengine=2.2.0
```

**建议: 可以通过label标记项目仓库地址，维护人联系方式、依赖的版本等信息**

### RUN

最常见的应该是安装软件包和一些shell命令，如 RUN apk --no-cache add curl git ....，我们可以通过 \ 分隔成多行便于查看软件列表的变更。如

```shell
apk --no-cache update && \
    apk --no-cache add curl \
        git \
        docker \
        nodejs 
```

**建议：多个脚本命令通过&&符号写在一个run指令中，可以有效减少layer数量。通过""分割行，可以方便预览命令变更。**

### CMD

如果你的镜像用于中间件 Server，CMD 的形式一般都是 CMD \[“executable”, “param1”, “param2”…]。如 CMD \["apache2","-DFOREGROUND"]。

CMD 还在大多数情况以交互式的方式出现。如 CMD \["python"]，当你执行 docker run -it python 的时候，将进入 shell 的交互模式。

CMD 很少以 CMD \[“param”, “param”] 协同 ENTRYPOINT 工作，除非我们很清楚它们俩的运行机制。

### EXPOSE

指定容器侦听端口，应该尽量使用应用程序通用的传统端口，如 Apache Web 服务器使用 EXPOSE 80 等。这个指令只是声明标记，具体不会在创建容器时被应用。

**建议: dockerfile中通过EXPOSE 标记服务会listen的端口**

### ENV

为容器添加环境变量，常用于为应用程序提供必要的环境变量以及版本号的设置，如：

{% code overflow="wrap" %}
```shell
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```
{% endcode %}

### ADD or COPY

这两者很相似，这里建议优先选择COPY，它比ADD透明度更高。

COPY只支持将本地文件复制到容器中。ADD除了COPY的功能外，还支持远程URL下载。但最好的用途是将本地tar文件提取到镜像中比如:

```dockerfile
ADD rootfs.tar.xz /
```

如果在Dockerfile中使用不用的文件，那么COPY它们可以单独使用。这样，特定文件的更改，将确保每一步的构建缓存无效，如：

```dockerfile
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```

将COPY . /tmp/ 放在后面，这能够使 RUN 的缓存无效的数量减少。

因为镜像大小很重要，故用 ADD 远程 URL 提取包是不被鼓励的，因该使用 curl 或 wget 替代。这样，能够减小镜像的层数。例如，你应该避免这样做：

```dockerfile
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

而是：

```dockerfile
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

对于不需要ADD tar 自动提取功能的其他项目（文件，目录），我们应该始终使用 COPY。

### ENTRYPOINT

ENTRYPOINT 的最好的用途时设置镜像的主命令，用 CMD 作为参数，这样就可以是镜像像命令一样运行，在容器启动时，我们只需要覆盖CMD参数即可。如:

CMD和ENTRYPOINT共存时，CMD会被当成ENTRYPOINT的参数传入,比如:

```dockerfile
ENTRYPOINT ["python"]
CMD ["--help"]
```

当容器运行时候我们可以覆盖CMD指令，比如:

```shell
docker run python -c "import sys ;print sys.version"

#-c "import sys ;print sys.version" 会覆盖掉--help参数。
```

ENTRYPOINT 也可以于脚本组合使用，如一个 Postgres Official Image 的例子。：

```shell
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

注：此脚本使用的 exec bash 命令 ，使最终运行的应用程序成为容器的 PID 1。这允许应用程序接收发送到容器任何 Unix 信号。将脚本复制到容器，并通过 ENTRYPOINT 开始运行：

```dockerfile
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
```

此脚本允许用户以多种方式与 Postgres 进行交。它可以简单地启动Postgres：

$ docker run postgres\
或者，它可以用于运行 Postgres 并将参数传递给服务器：

```shell
$ docker run postgres postgres --help
```

它也可以用来启动一个完全不同的工具，比如 Bash：

```shell
$ docker run --rm -it postgres bash
```

**建议在业务的dockerfile中用entrypoint作为镜像的主命令，CMD作为主命令默认的flag**

### VOLUME

VOLUME 指令应该用于如下内容：任何类型的数据库存储区域、配置存储、容器创建的文件或目录。\
推荐 VOLUME 用于挂载镜像中那些经常变化（易变化的）或者用户可维护的部分。

USER\
如果一个服务不需要超级权限来运行，我们可以通过 USER 切换成非 root 用户。在 Dockerfile 中用如下方式创建：

```shell
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres
USER postgres
```

**建议为了减少层数和复杂度，避免频繁使用 USER 进行用户切换**

### **WORKDIR**

我们使用WROKDIR作为工作路径。而不是增加复杂的命令，如 RUN cd … && do-something这样难以阅读以及排障困难和难以维护。

```shell
COPY cli /data/apps/bin/
WORKDIR /data/apps/bin
ENTRYPOINT ["cli"]
```





## 3、安装

### 1、centos7安装docker

{% code overflow="wrap" lineNumbers="true" fullWidth="false" %}
```sh
# 安装docker依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

# 配置docker yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 查看可安装版本
yum list docker-ce --showduplicates | sort -r

# 安装
yum -y install docker-ce-18.03.1.ce
```
{% endcode %}

### 2、docker-compose 安装

{% code overflow="wrap" fullWidth="false" %}
```shell
# 下载
curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 加权限
chmod +x /usr/local/bin/docker-compose

# add bin
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

```
{% endcode %}

### 3、ubuntu 安装

{% code overflow="wrap" fullWidth="false" %}
```sh
apt-get update
apt-get -y install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
apt-get -y update
#查看可安装版本
apt-cache madison docker-ce
#指定版本安装
apt-get -y install docker-ce=5:19.03.14~3-0~ubuntu-bionic
system enable docker
```
{% endcode %}

### 4、docker 配置文件

{% code fullWidth="false" %}
```sh
cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-opts": {"max-size": "100m"},
  "debug" : true,
  "experimental" : true,
  "registry-mirrors": ["https://mirror.ccs.tencentyun.com",
                        "http://b611224e.m.daocloud.io",
                        "http://fa42fdd5.m.daocloud.io",
                        "https://odhacnra.mirror.aliyuncs.com"],
  "storage-driver": "overlay",
  "max-concurrent-downloads": 10,
  "live-restore": true
}
EOF
```
{% endcode %}
