sudo passwd root

su root

vim /etc/wsl.conf

```bash
# 启动设置
[boot]
# 提供systemd支持，以便能够使用systemctl等命令
systemd=true

# 装载设置
[automount]
# 禁止挂载Windows磁盘（C盘D盘等挂载到/mnt/c, /mnt/d等目录），防止影响文件的索引速度和权限
enabled=false

# 互操作设置
[interop]
# 禁止启动windows进程
enabled=false
# 禁止将Windows的环境变量PATH引入到Linux的环境变量PATH里
appendWindowsPath=false

# 用户设置
[user]
# 启动后默认登录用户
default=root
```

vim ~/.bashrc

```bash
export PS1='\e[1;32m\u@\h\e[0m:\e[1;34m\w\e[0m\$ '

alias c='clear'

alias u='apt update'
alias g='apt upgrade'
alias i='apt install'
alias r='apt remove'
alias ar='apt autoremove'

alias p='ps -ef | grep'
alias n='netstat -tulp | grep'
```

timedatectl
timedatectl set-timezone Asia/Shanghai

hostnamectl
hostnamectl --static set-hostname [自定义主机名]

cd /etc/apt
vim sources.list.tsinghua

```bash
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# # deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

cp sources.list sources.list.bak && \
cp sources.list.tsinghua sources.list

apt update && \
apt upgrade

vim /etc/ld.so.conf.d/custom.conf

```bash
/usr/lib
/usr/lib64
/usr/local/lib
/usr/local/lib64
```

ldconfig
