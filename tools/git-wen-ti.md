# 🐶 Git

### 1、git 仓库过大，无法直接clone

```
首先关闭 core.compression

git config --global core.compression 0
然后使用depth这个指令来下载最近一次提交

git clone --depth 1 url
然后获取完整库

git fetch --unshallow 
最后pull一下查看状态，问题解决

git pull --all
```

### 2、不用重复输入账号密码

`git config --global credential.helper store`

### 3、实现linux终端显示当前分支

```bash
vim .bashrc

# Show git branch name
force_color_prompt=yes
color_prompt=yes
parse_git_branch() {
 git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}
if [ "$color_prompt" = yes ]; then
 PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[01;31m\]$(parse_git_branch)\[\033[00m\]\$ '
else
 PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w$(parse_git_branch)\$ '
fi
unset color_prompt force_color_prompt

source .bashrc
```
