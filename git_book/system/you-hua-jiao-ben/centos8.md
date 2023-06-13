# centos8

#### bash 自动补全

```bash
yum install bash-completion -y
```

#### shell终端显示日期、路径

```bash
echo "export PS1='[` date +"%Y-%m-%d %T"` \u@\H \w]# '" > /etc/profile
```

#### 显示历史记录时间

```bash
export HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S  "
```

