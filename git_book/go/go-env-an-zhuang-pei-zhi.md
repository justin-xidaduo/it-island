# go 安装配置

## 一、MAC 安装配置

### 1、安装go

从 [https://go.dev/dl/](https://go.dev/dl/) 下载合适版本，根据安装指导进行安装。默认安装位置为 `/usr/local/go`

### 2、配置GOROOT、GOPATH、GOBIN

```sh
# 编辑.zshrc
vim ~/.zshrc

export GOROOT=/usr/local/go
export GOPATH=/Users/zhaolibin/go
export GOBIN=$GOPATH/bin/
export PATH=$PATH:$GOPATH/bin

# check
go env
```







##

## 1、centos7 安装配置go环境

### 1、go环境变量配置 (GOROOT和GOPATH)

#### GOROOT 就是go的安装路径(配置如下)

```
# 下载golang
# 官网 https://golang.google.cn/dl/
cd /data/src
wget https://golang.google.cn/dl/go1.15.1.linux-amd64.tar.gz -O /data/src/go1.15.1.linux-amd64.tar.gz
mv go go1.15.1
mv go1.15.1 /usr/local/
ln -s /usr/local/go1.15.1 /usr/local/go

# vim /etc/profile

```

#### GOPATH

* `go install/go get`和 go的工具等会用到GOPATH环境变量。
* GOPATH是作为编译后二进制的存放目的地和import包时的搜索路径 (其实也是你的工作目录, 你可以在src下创建你自己的go源文件, 然后开始工作)。
* 不要把GOPATH设置成go的安装路径

1. GOPATH之下主要包含三个目录: bin、pkg、src
2. bin目录主要存放可执行文件; pkg目录存放编译好的库文件, 主要是\*.a文件; src目录下主要存放go的源文件

```
mkdir /opt/gopath

export GOROOT=/usr/local/go
export PATH=.:$GOROOT/bin:$PATH

export GOPATH=/opt/gopath
export PATH=.:$GOPATH/bin:$PATH
```

### 配置插件等

#### 方法1

```
# 创建github.com插件目录及下载插件
cd $GOPATH/src
mkdir github.com
cd $GOPATH/src/github.com
mkdir acroca cweill derekparker go-delve josharian karrick mdempsky pkg ramya-rao-a rogpeppe sqs uudashr
cd $GOPATH/src/github.com/acroca
git clone https://github.com/acroca/go-symbols.git
cd $GOPATH/src/github.com/cweill
git clone https://github.com/cweill/gotests.git
cd $GOPATH/src/github.com/derekparker
git clone https://github.com/derekparker/delve.git
cd $GOPATH/src/github.com/go-delve
git clone https://github.com/go-delve\delve.git
cd $GOPATH/src/github.com/josharian
git clone https://github.com/josharian/impl.git
cd $GOPATH/src/github.com/karrick
git clone https://github.com/karrick/godirwalk.git
cd $GOPATH/src/github.com/mdempsky
git clone https://github.com/mdempsky/gocode.git
cd $GOPATH/src/github.com/pkg
git clone https://github.com/pkg/errors.git
cd $GOPATH/src/github.com/ramya-rao-a
git clone https://github.com/ramya-rao-a/go-outline.git
cd $GOPATH/src/github.com/rogpeppe
git clone https://github.com/rogpeppe/godef.git
cd $GOPATH/src/github.com/sqs
git clone https://github.com/sqs/goreturns.git
cd $GOPATH/src/github.com/uudashr
git clone https://github.com/uudashr/gopkgs.git

# 创建golang.org插件目录及下载插件
cd $GOPATH/src
mkdir -p golang.org/x
cd golang.org/x
git clone https://github.com/golang/tools.git
git clone https://github.com/golang/lint.git
git clone https://github.com/golang/mod.git
git clone https://github.com/golang/xerrors.git

# 手动安装插件
cd $GOPATH/src
go install github.com/mdempsky/gocode
go install github.com/uudashr/gopkgs/cmd/gopkgs
go install github.com/ramya-rao-a/go-outline
go install github.com/acroca/go-symbols
go install github.com/rogpeppe/godef
go install github.com/sqs/goreturns
go install github.com/derekparker/delve/cmd/dlv
go install github.com/cweill/gotests
go install github.com/josharian/impl
go install golang.org/x/tools/cmd/guru
go install golang.org/x/tools/cmd/gorename
go install golang.org/x/lint/golint

```

#### 方法2

```
go env -w  GOPROXY=https://goproxy.cn,direct

go install github.com/mdempsky/gocode
go install github.com/uudashr/gopkgs/cmd/gopkgs
go install github.com/ramya-rao-a/go-outline
go install github.com/acroca/go-symbols
go install github.com/rogpeppe/godef
go install github.com/sqs/goreturns
go install github.com/derekparker/delve/cmd/dlv
go install github.com/cweill/gotests
go install github.com/josharian/impl
go install golang.org/x/tools/cmd/guru
go install golang.org/x/tools/cmd/gorename
go install golang.org/x/lint/golint


go get -u -v github.com/ramya-rao-a/go-outline
go get -u -v github.com/acroca/go-symbols
go get -u -v github.com/mdempsky/gocode
go get -u -v github.com/rogpeppe/godef
go get -u -v golang.org/x/tools/cmd/godoc
go get -u -v github.com/zmb3/gogetdoc
go get -u -v golang.org/x/lint/golint
go get -u -v github.com/fatih/gomodifytags
go get -u -v golang.org/x/tools/cmd/gorename
go get -u -v sourcegraph.com/sqs/goreturns
go get -u -v golang.org/x/tools/cmd/goimports
go get -u -v github.com/cweill/gotests/...
go get -u -v golang.org/x/tools/cmd/guru
go get -u -v github.com/josharian/impl
go get -u -v github.com/haya14busa/goplay/cmd/goplay
go get -u -v github.com/uudashr/gopkgs/cmd/gopkgs
go get -u -v github.com/davidrjenni/reftools/cmd/fillstruct
```
