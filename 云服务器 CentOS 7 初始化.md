# 云服务器 CentOS 7 初始化

初始化时，建议使用 root 用户，以便全局修改

## 设置 SSH 连接有效时长

默认设置的太短了，稍微不动终端就断连，所以加长点连接时间

```bash
cd /etc/ssh
vim sshd_config
```

修改这两项

```bash
# 心跳检测间隔时间，单位秒
ClientAliveInterval 60
# 客户端没有活动的最大时长，单位秒
ClientAliveCountMax 3600
```

修改完保存并重启 SSH 服务

```bash
systemctl restart sshd
```

## 修改主机名

```bash
hostnamectl set-hostname 你的主机名
```

修改完需要重连 SSH 才会生效

## 美化终端样式

修改用户级终端配置

```bash
cd ~
vim ./bash_profile
```

在文件添加

```bash
export PS1='[\e[1;32m\u@\h\e[0m \e[1;34m\W\e[0m]\$ '
```

## 添加快捷指令

修改用户级终端配置

```bash
cd ~
vim .bashrc
```

在文件添加

```bash
alias c='clear'
alias la='ls -al'

alias u='yum update'
alias i='yum install'
alias r='yum remove'

alias p='ps -ef | grep'
alias n='netstat -tulp | grep'

alias pl='pkg-config --list-all | grep'
alias pm='pkg-config --modversion'
alias lg='ldconfig -p | grep'
```

修改完用户级终端配置后，需要刷新终端
在 CentOS 里，刷新 profile 时会同步刷新 bashrc，正好两步修改完一并刷新

```bash
source ~/.bash_profile
```

## yum 开启包缓存

yum 安装完软件包后，会删除缓存，下次再装还需要下载，这里修改配置，保留缓存

```bash
vim /etc/yum.conf
```

将 keepcache 的值改为 1 表示保留缓存

## yum 关闭比较镜像源速度

每次下载安装前会比较最快的镜像源，有时安装很小的软件包，比较花费时长甚至比安装时长还多得多，所以可以考虑关闭

```bash
cd /etc/yum/pluginconf.d
vim fastestmirror.conf
```

将 enabled 改为 0 表示禁用

## 更新软件包

上面已经添加过快捷指令了，这里直接执行

```bash
# yum update
u
```

## 添加动态链接库路径

ldconfig 用来管理系统动态链接库，现在配置好所有链接库路径，省得以后安装时出现一些琐碎麻烦

新建一个 conf 文件保存

```bash
cd /etc/ld.so.conf.d
vim custom.conf
```

写上四处 lib 路径并保存

```bash
/usr/lib
/usr/lib64
/usr/local/lib
/usr/local/lib64
```

最后执行 ldconfig 刷新库

## 添加 pkg-config 管理的模块包路径

pkg-config 管理的 C++模块包，也有一些常用的存放路径

在系统级终端配置 profile 里添加环境变量，最好新建一个，不和系统自带的混淆

```bash
cd /etc/profile.d
vim custom.sh
```

在文件里添加

```bash
# pkg-config 管理的共享库路径
if [ ! $PKG_CONFIG_PATH] then
    export $PKG_CONFIG_PATH=/usr/lib/pkgconfig:/usr/lib64/pkgconfig:/usr/local/lib/
else
    export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig:/usr/lib64/pkgconfig:/usr/local/lib/
fi
```

刷新终端

```bash
source /etc/profile
```
