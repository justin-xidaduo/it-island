# 安装配置

```
关闭防火墙
/etc/init.d/ufw stop

配置hosts以及免密登录
172.17.0.101 iot-server1
172.17.0.102 iot-server2
172.17.0.103 iot-server3

内核优化参数  /etc/sysctl.conf
kernel.shmmax = 500000000
kernel.shmmni = 4096
kernel.shmall = 4000000000
kernel.sem = 250 512000 100 2048
kernel.sysrq = 1
kernel.core_uses_pid = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.conf.all.arp_filter = 1
net.ipv4.ip_local_port_range = 1025 65535
net.core.netdev_max_backlog = 10000
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.overcommit_memory = 2
vm.overcommit_ratio =50
修改限制文件数   /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
增加源，安装

apt-get install python-software-properties



add-apt-repository ppa:greenplum/db
apt-get update
apt-get install greenplum-db-5


用户
 groupadd -g 530 gpadmin
useradd -g 530 -u 530 -m -d /home/gpadmin -s /bin/bash gpadmin

chown -R gpadmin:gpadmin /home/gpadmin/
passwd gpadmin


安装
pip install --upgrade pip
sudo -H pip uninstall paramiko
sudo -H pip uninstall pycrypto
sudo -H pip install paramiko

# cat all_hosts
iot-server1
iot-server2
iot-server3

root@iot-server1:/opt/gpdb# cat seg_hosts
iot-server2
iot-server3

su - gpadmin
cd /opt/gpdb/
source greenplum_path.sh
gpssh-exkeys -f all_hosts   #设置免密
gpssh -f all_hosts -e ls $GPHOME #验证

# 安装
gpseginstall -f all_hosts -u gpadmin

mkdir -p /data/greenplum/master/
chown gpadmin /data/greenplum/master/

gpssh -f seg_hosts -e 'mkdir -p /data/greenplum/primary'
gpssh -f seg_hosts -e 'mkdir -p /data/greenplum/mirror'
gpssh -f seg_hosts -e 'chown gpadmin /data/greenplum/primary'
gpssh -f seg_hosts -e 'chown gpadmin /data/greenplum/mirror'


# 时间同步
apt-get install ntpd -y

# 验证系统环境
gpcheck -f all_hosts -m iot-server1

# 初始化
su - gpadmin 
cp /opt/gpdb/docs/cli_help/gpconfigs/gpinitsystem_config ~

chmod 755 gpinitsystem_config


gpadmin@iot-server1:~$ cat gpinitsystem_config | grep -v ^$ |grep -v "#"
ARRAY_NAME="Greenplum Data Platform"
SEG_PREFIX=gpseg
PORT_BASE=40000
declare -a DATA_DIRECTORY=(/data/greenplum/primary)
MASTER_HOSTNAME=iot-server1
MASTER_DIRECTORY=/data/greenplum/master
MASTER_PORT=5432
TRUSTED_SHELL=ssh
CHECK_POINT_SEGMENTS=8
ENCODING=UNICODE
declare -a MIRROR_DATA_DIRECTORY=(/data/greenplum/mirror)

#开始初始化
gpinitsystem -c gpinitsystem_config -h /opt/gpdb/seg_hosts

# 加入.bashrc (三台机器都做操作)
export MASTER_DATA_DIRECTORY=/data/greenplum/master/gpseg-1


$ gpstart

```

