# NFS

ubuntu 16.04 安装NFS

```bash
# NFS Server
apt install nfs-kernel-server -y
echo "/nfs_data 172.16.10.0/24(rw,sync,no_subtree_check,no_root_squash)" >> /etc/exports

systemctl restart nfs-kernel-server
systemctl enable nfs-kernel-server

# NFS Client
apt-get install -y nfs-common -y 
mkdir /nfs_data
mount -t nfs 172.16.10.11:/nfs_data /nfs_data

172.16.10.11:/nfs_data    /nfs_data      nfs        defaults,_rnetdev     0 0
```
