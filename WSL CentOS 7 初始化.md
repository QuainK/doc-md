# WSL CentOS 7 初始化

设置网络适配器为 VMNet 8 NAT 模式

设置 root 密码

语言 en-US
时区 Asia/Shanghai

```bash
timedatectl
timedatectl set-timezone Asia/Shanghai
```

查看网卡

```bash
ip a

cd /etc/sysconfig/network-scripts
vi ifcfg-ens33
```

ONBOOT=yes

```bash
systemctl restart network
```

开启 ssh 登录

```bash
cd /etc/ssh
cp sshd_config sshd_config.bak
vi /etc/ssh/sshd_config
```

- ListenAddress 0.0.0.0
- ListenAddress ::
- PermitRootLogin yes
- PasswordAuthentication yes
- UseDNS no

```bash
systemctl restart sshd
```

```bash
vi /etc/profile.d/custom.sh
```

```bash
# pkg-config 管理的共享库路径
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig:/usr/lib64/pkgconfig:/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig
# 系统共享库路径，这里不需要，只需要在/etc/ld.so.conf.d/里创建文件
#export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib:/usr/lib64:/usr/local/lib:/usr/local/lib64
# 启用 gcc 7.x
#scl enable devtoolset-7 bash
```

```bash
vi /etc/ld.so.conf.d/custom.conf
```

```bash
/usr/lib
/usr/lib64
/usr/local/lib
/usr/local/lib64
```

```bash
ldconfig
```

```bash
vi ~/.bash_profile
```

```bash
export PS1='[\e[1;32m\u@\h\e[0m \e[1;34m\w\e[0m]\$ '
```

```bash
vi ~/.bashrc
```

```bash
alias c='clear'

alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'

alias u='yum update'
alias i='yum install'
alias r='yum remove'

alias p='ps -ef | grep'
alias n='netstat -tulp | grep'
```

```bash
source /etc/profile
source ~/.bash_profile
```

```bash
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/centos|baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
```

```bash
vi /etc/yum/pluginconf.d/fastestmirror.conf
```

```bash
enabled=0
```

```bash
vi /etc/yum.conf
```

```bash
keepcache=1
```

```bash
yum clean all
yum makecache
yum install epel-release

sed -e 's!^metalink=!#metalink=!g' \
    -e 's!^#baseurl=!baseurl=!g' \
    -e 's!https\?://download\.fedoraproject\.org/pub/epel!https://mirrors.tuna.tsinghua.edu.cn/epel!g' \
    -e 's!https\?://download\.example/pub/epel!https://mirrors.tuna.tsinghua.edu.cn/epel!g' \
    -i /etc/yum.repos.d/epel*.repo
yum clean all
yum makecache

yum update

yum install vim
yum install wget

yum provides \*/ifconfig
yum install net-tools
```
