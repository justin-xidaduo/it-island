# k8s 系统初始化 基础调优

#### 1.完成系统初始化和基本调优，参考[ centos7系统初始化](https://it.zhaolibin.com/system/you-hua-jiao-ben/system\_init)

#### 2.k8s安装的系统初始化和调优

```
# 关闭swap
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# 内核参数
cat > kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1 
net.bridge.bridge-nf-call-ip6tables=1 
net.ipv4.ip_forward=1 
net.ipv4.tcp_tw_recycle=0 
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_instances=8192 
fs.inotify.max_user_watches=1048576 
fs.file-max=52706963 
fs.nr_open=52706963 
net.ipv6.conf.all.disable_ipv6=1 
net.netfilter.nf_conntrack_max=2310720
EOF
cp kubernetes.conf  /etc/sysctl.d/kubernetes.conf 
sysctl -p /etc/sysctl.d/kubernetes.conf

# 设置 rsyslogd 和 systemd journald
mkdir /var/log/journal # 持久化保存日志的目录 
mkdir /etc/systemd/journald.conf.d 
cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal] 
# 持久化保存到磁盘 
Storage=persistent 
# 压缩历史日志 
Compress=yes 
SyncIntervalSec=5m 
RateLimitInterval=30s 
RateLimitBurst=1000 
# 最大占用空间    10G 
SystemMaxUse=10G
# 单日志文件最大    200M
SystemMaxFileSize=200M 
# 日志保存时间    2 周 
MaxRetentionSec=2week 
# 不将日志转发到    syslog 
ForwardToSyslog=no 
EOF

systemctl restart systemd-journald

# 关闭NUMA
cp /etc/default/grub{,.bak} 
vim /etc/default/grub # 在    GRUB_CMDLINE_LINUX 一行添加    `numa=off` 参数，如下所示： diff /etc/default/grub.bak /etc/default/grub 
6c6 
< GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rhgb quiet" 
--- 
> GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rhgb quiet numa=off" 
cp /boot/grub2/grub.cfg{,.bak} 
grub2-mkconfig -o /boot/grub2/grub.cfg
```
