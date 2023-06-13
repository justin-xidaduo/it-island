# install istio-1.6.5

### 下载二进制包

官网地址 [https://github.com/istio/istio/releases/tag/1.6.5](https://github.com/istio/istio/releases/tag/1.6.5)

```
curl -L https://istio.io/downloadIstio  | sh -
cd /data/istio/src
wget https://github.com/istio/istio/releases/download/1.6.5/istioctl-1.6.5-linux-amd64.tar.gz
# wget http://it2.zhaolibin.com/src/istio/istio-1.6.5-linux-amd64.tar.gz

# 解压
tar -zxvf istio-1.6.5-linux-amd64.tar.gz

# 拷贝二进制文件
cp /data/istio/src/istio-1.6.5/bin/istioctl /usr/local/bin/
```

### 配置自动补全

```
cp /data/istio/src/istio-1.6.5/tools/istioctl.bash ~/.istioctl.bash
echo "source ~/.istioctl.bash" >> ~/.bash_profile
source ~/.bash_profile
```

