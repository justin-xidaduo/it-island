# 🐰 linux

## 1、bash相关

### bash 自动补全

```bash
yum install bash-completion -y
```

### shell终端显示日期、路径

```bash
echo "export PS1='[` date +"%Y-%m-%d %T"` \u@\H \w]# '" > /etc/profile
```

### 显示历史记录时间

```bash
export HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S  "
```

## 2、ssh 配置

```bash
#设置超时退出 #设置300*100秒
ClientAliveInterval 300
ClientAliveCountMax 100

#允许root登录
PermitRootLogin yesbash
```
