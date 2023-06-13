# ubuntu 16.04 安装

```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

sudo curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

sudo tee /etc/apt/sources.list.d/kubernetes.list <<-'EOF'
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main
EOF


sudo apt-get update

apt-get install kubeadm=1.18.2-00 kubelet=1.18.2-00 kubectl=1.18.2-00
apt-mark hold  kubeadm=1.18.2-00 kubelet=1.18.2-00 kubectl=1.18.2-00

systemctl enable kubelet && sudo systemctl start kubelet


```
