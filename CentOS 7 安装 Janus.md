# CentOS 7 安装 Janus

## 〇、整体流程

1. 准备 Janus 依赖
2. 安装 Janus
3. 测试 Janus 命令可用
4. 安装 coturn
5. 准备 SSL 证书
6. 配置并启动 coturn
7. 配置 Janus
8. 配置外网防火墙，开启对外开放的端口
9. 启动 Janus 守护进程（可选：手动创建服务并设置开机自启）
10. 外网端口测试

## 一、准备 Janus 依赖

先用包管理器尽量安装好各种依赖，包管理器搜不到这个包或者能装的最高版本也不符合时，再依次从依赖包的官网或者 Git
仓库下载源码，进行本地编译安装。

---

- 系统平台：CentOS 7.9 x64
- 包管理器：yum 3.4.3

---

Janus 官方 Github，使用了 1.1.4 版本，参考官方 README.md 里的依赖安装流程

<https://github.com/meetecho/janus-gateway/tree/v1.1.4>

---

### 0. yum install xxx

执行此条命令，一键安装大部分依赖

```bash
yum install libmicrohttpd-devel jansson-devel \
 openssl-devel libsrtp-devel sofia-sip-devel glib2-devel \
 opus-devel libogg-devel libcurl-devel pkgconfig gengetopt \
 libconfig-devel libtool autoconf automake
```

在 CentOS 7 里能用 yum 直接安装好的依赖有：

- jansson-devel
- openssl-devel
- glib2-devel
- opus-devel
- libogg-devel
- libcurl-devel
- pkgconfig
- gengetopt
- libconfig-devel
- libtool
- autoconf
- automake

剩下的一些都要源码安装：

- nettle
- gnutls
- libmicrohttpd
- libsrtp
- libnice
- libwebsockets

---

### 设置 pkgconfig 环境变量

```bash
PKG_CONFIG_PATH=/usr/lib/pkgconfig:/usr/lib64/pkgconfig:/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig
```

在/etc/ld.so.conf.d 里新建文件 custom.conf 添加几行

### 将 gcc 升级到 7.x 版本

```bash
# 安装工具包
yum install centos-release-scl
yum install devtoolset-7-gcc-c++

# 修改~/.bashrc或者/etc/profile使之永久生效
# 使用此句
# scl enable devtoolset-7 bash
# 或者此句
# 就用此局，上面那句会出现死循环登不进shell
source /opt/rh/devtoolset-7/enable
```

### 准备 meson ninja

记得给 pip3 配置国内镜像源

```bash
pip3 install meson
pip3 install ninja
```

### 1. nettle

GNU 密码库

#### 下载地址

<https://ftp.gnu.org/gnu/nettle/nettle-3.4.1.tar.gz>

#### 版本

3.4.1

#### 必需依赖

- libgmp (libhogweed)

#### 编译安装过程

用 make 编译

```bash
wget https://ftp.gnu.org/gnu/nettle/nettle-3.4.1.tar.gz
tar -xvf nettle-3.4.1.tar.gz
# 进入源码根目录
cd nettle-3.4.1
# 生成编译配置信息Makefile
# 设置安装路径 /usr/local
./configure --prefix=/usr/local
# 编译
make
# 安装
# 安装的文件存放在 /usr/local/xxx 目录
make install
# 生效共享库
ldconfig
```

#### 卸载

```bash
make uninstall
```

---

### 2. gnutls

GNU 安全通信库（SSL、TLS、DTLS 等）

#### yum 安装方式

```bash
yum install gnutls-devel
```

#### 源码文件下载地址

<https://www.gnupg.org/ftp/gcrypt/gnutls/v3.5/>

#### 版本

3.5.19

#### 必需依赖

- nettle
- libgmp (libhogweed)

#### 编译安装过程

用 make 编译

```bash
# 进入源码根目录
cd gnutls-3.5.19
# 生成编译配置信息Makefile
# 设置安装路径 /usr/local
# 包含和排除引入某些库
# ./configure --prefix=/usr/local --with-included-libhogweed --with-included-libtasn1 --with-included-unistring --without-p11-kit
./configure --prefix=/usr/local --with-included-libtasn1 --with-included-unistring --without-p11-kit
# 编译
make
# 安装
# 安装的文件存放在 /usr/local/xxx 目录
make install
# 生效共享库
ldconfig
# 版本号
pkg-config --modversion nettle
# 输出 3.4.1
```

#### 卸载

```bash
make uninstall
```

---

### 3. libmicrohttpd

GNU HTTP 1.1 服务器

#### 下载地址

<https://ftp.gnu.org/gnu/libmicrohttpd/>

#### 版本

0.9.77

#### 前提依赖

HTTPS 支持需要 libgnutls

#### 编译安装过程

用 make 编译

```bash
# 进入源码根目录
cd libmicrohttpd-0.9.77
# 生成编译配置信息Makefile
# 设置安装路径 /usr/local
# 启用HTTPS，启用SSL
./configure --prefix=/usr/local --enable-https=yes --with-ssl
# 编译
make
# 安装
# 安装的文件存放在 /usr/local/xxx 目录
make install
# 生效共享库
ldconfig
```

#### 卸载

```bash
make uninstall
```

---

### 4. libsrtp

Cisco 安全实时传输协议（Secure Real-Time Transfer Protocol）

#### git 仓库

<https://github.com/cisco/libsrtp/tree/v2.5.0>

#### 版本

2.5.0

#### 编译安装过程

需要 epel-release CMake3、libpcap、doxygen

cmake3 软链接到 cmake

用 Meson 构建，用 Ninja 编译

```bash
wget -O libsrtp-2.5.0.zip https://codeload.github.com/cisco/libsrtp/zip/refs/tags/v2.5.0
# 进入源码根目录
cd libsrtp-2.5.0
# 生成构建依赖信息，生成编译目录 build
meson setup build
# 进入编译目录
cd build
# 编译（ninja不带路径默认当前目录，-C [path] 指定路径）
ninja
# 测试
ninja test
# 安装
# 安装的文件存放在 /usr/local/xxx 目录
# 安装后会自动生成安装记录 ./meson-logs/install-log.txt，不要删除，卸载时需要参考，如果删除了，重新安装一遍就行
ninja install
# 生效共享库
ldconfig
```

#### 卸载

```bash
cd build
ninja uninstall
```

---

### 5. libnice

GLib ICE Library，GLib 的一个用于 NAT 穿透的库（ICE 是 NAT 穿透的一种框架）

#### 下载地址

<https://codeload.github.com/libnice/libnice/zip/refs/tags/0.1.21>

#### 注意

yum 不要安装对应包（\*-devel），否则会有版本冲突

#### 编译安装过程

用 Meson 构建，用 Ninja 编译

```bash
# 进入源码根目录
cd libnice-0.1.18
# 生成构建依赖信息，生成编译目录 build
meson  build
# 进入编译目录
cd build
# 编译（ninja不带路径默认当前目录，-C [path] 指定路径）
ninja
# 测试
ninja test
# 安装
# 安装的文件存放在 /usr/local/xxx 目录
# 安装后会自动生成安装记录 ./meson-logs/install-log.txt，不要删除，卸载时需要参考，如果删除了，重新安装一遍就行
ninja install
# 生效共享库
ldconfig
```

#### 卸载

```bash
cd build
ninja uninstall
```

---

### 6. libwebsockets

Websocket 通信支持

#### git 仓库

<https://github.com/warmcat/libwebsockets/archive/refs/tags/v4.3.2.zip>

#### 版本

4.3.2

#### 编译安装过程

用 CMake 构建，用 make 编译

```bash
# 进入源码根目录
cd libwebsockets-4.3.2
# 创建编译目录 build
mkdir build
# 进入编译目录
cd build
# 生成构建依赖信息
# 设置安装路径 /usr/local
# 设置编译标志 fPIC 生成位置无关代码（方便共享库文件，文件使用相对路径进行跳转和链接等）
# .. 源码根目录
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr/local -DCMAKE_C_FLAGS="-fpic" ..
# 编译
make
# 安装
# 安装的文件存放在 /usr/local/xxx 目录
# 安装后会自动生成安装记录 ./install_manifest.txt，不要删除，卸载时需要参考，如果删除了，重新安装一遍就行
make install
# 生效共享库
ldconfig
```

#### 卸载

```bash
cd build
# 因为Makefile没有提供卸载指令，所以从安装记录里手动卸载
xargs rm < install_manifest.txt
```

### 7. sofia-sip

#### 安装手册

<https://github.com/freeswitch/sofia-sip/blob/master/README>

#### 安装步骤

```bash
wget -O sofia-sip-1.13.15.zip https://github.com/freeswitch/sofia-sip/archive/refs/tags/v1.13.15.zip
unzip sofia-sip-1.13.15.zip
cd sofia-sip-1.13.15
sh autogen.sh (if building from darcs)
./configure
make
make install
make uninstall
ldconfig
```

### 8. usrsctp

#### 安装手册

<https://github.com/sctplab/usrsctp/blob/master/Manual.md>

#### 安装步骤

##### 法一、make

```bash
wget -O usrsctp-0.9.5.0.zip https://github.com/sctplab/usrsctp/archive/refs/tags/0.9.5.0.zip
unzip usrsctp-0.9.5.0.zip
cd usrsctp-0.9.5.0
./bootstrap
./configure
make
make install
ldconfig
```

##### 法二、cmake

```bash
wget -O usrsctp-0.9.5.0.zip https://github.com/sctplab/usrsctp/archive/refs/tags/0.9.5.0.zip
unzip usrsctp-0.9.5.0.zip
cd usrsctp-0.9.5.0
mkdir build
cd build
cmake ..
make
make install
ldconfig
```

---

## 二、安装 Janus

### 1. 源码安装 janus-gateway

安装本体，Janus 服务器

#### git 仓库

<https://github.com/meetecho/janus-gateway/tree/v1.1.4>

#### 版本

1.1.4

#### 依赖

需要 gcc 7

#### 编译安装过程

用 make 编译

```bash
wget -O janus-gateway-1.1.4.zip https://github.com/meetecho/janus-gateway/archive/refs/tags/v1.1.4.zip
unzip janus-gateway-1.1.4.zip
# 进入源码根目录
cd janus-gateway-1.1.4
# 生成编译配置信息Makefile
sh autogen.sh
# 设置安装路径 /opt/janus
# 启用HTTP和HTTPS
# 启用WebSocket
# 启用srtp 2.x （用于SIP电话）
./configure --prefix=/opt/janus --enable-rest --enable-websockets --enable-libsrtp2
#./configure --prefix=/opt/janus --enable-websockets --enable-libsrtp2
# 编译
make
# 安装
# 安装的文件存放在 /opt/janus 目录，里面有bin, etc, share, lib, include等子目录
make install
# 添加配置文件，会在/opt/janus/etc/janus/conf里自动生成默认配置，后续步骤需要进去修改
make configs
```

#### 卸载

```bash
make uninstall
```

---

### 2. 添加环境变量

在用户(~/.bashrc)或系统(/etc/profile)环境变量里添加 Janus 的 bin 目录，如果按上述流程安装，路径应该为

> /opt/janus/bin

比如在 ~/.bashrc 中添加

> PATH=$PATH:/opt/janus/bin

然后刷新环境变量

```bash
source ~/.bashrc
```

### 3. 测试 Janus 命令是否可用

```bash
janus --help
```

显示 Janus 版本信息和帮助信息

## 三、安装 coturn

用于 NAT 穿透的一个 STUN/TURN 服务器，给 Janus 服务器和客户端提供对等通信

coturn, turnserver, turnadmin 等命令都是此工具的命令

1. 通常 yum 可以直接安装

```bash
yum install coturn
```

## 四、准备 SSL 证书

### 1. 申请证书

找各大服务商在线申请，提供公司信息和域名，可能需要较长时间

### 2. 证书下载

将申请好的证书放入服务器某个路径，后续步骤的一些配置需要使用

## 五、配置 coturn

### 1. 修改 coturn 的 conf 文件

通常情况下，coturn 的配置文件位于

> /etc/coturn/turnserver.conf

其中需要改动的地方如下（行数仅供参考，不一定完全精确），其余保持默认即可

```
# 18行 监听端口号
listen-port=3478

# 60行 监听的IP，通常是指coturn的宿主服务器，可以同时配置多条
listen-ip=192.168.23.176

# 189行 无需验证即可使用
no-auth

# 362行 对外展示的名称，通常指服务器绑定的IP或者域名
realm='192.168.23.176'

# 464行 证书（公钥），仅支持pem格式（见461行官方注释提醒）
cert=/xxx/xxx/xxx.pem

# 472行 私钥
pkey=/xxx/xxx/xxx.key

# 477行 私钥密码（若需要）
pkey-pwd=xxx

# 533行 日志路径
log-file=/var/log/coturn/turnserver.log

# 662行 进程ID文件，用于进程锁，防止启动多个进程
pidfile="/var/run/turnserver.pid"

# 727行 命令行密码（目前用不到，只是需要配置）
cli-password=qwerty
```

### 2. 启动 coturn 服务

```bash
systemctl start coturn
systemctl status coturn
```

启动成功的话，Active 行会显示绿色的 <font color=green>active (running)</font> 字样
启动失败的话，可以去日志路径查看报错日志信息对症下药

可以将此服务设置成开机自启动

```bash
systemctl enable coturn
```

## 六、配置 Janus

Janus 需要配置的文件有三个，分别是：

- 通用全局配置（janus.jcfg）
- 针对 HTTP 和 HTTPS 通信的配置（janus.transport.http.jcfg）
- 针对 WebSocket 和 WebSockets 通信的配置（janus.transport.websockets.jcfg）

如果没有相关文件，可以从同名的 xxx.sample 默认文件复制过来

### 1. 修改通用全局配置 janus.jcfg

其中需要改动的地方如下（行数仅供参考，不一定完全精确），其余保持默认即可

```
# 16行 日志路径
log_to_file = "/var/log/janus/janus.log"

# 18行 日志启用时间记录，这样每条日志开头都有ISO格式的日期时间
debug_timestamps = true

# 19行 消息类型启用颜色区分，一般白，警告黄，错误红
debug_color = true

# 28行 启用守护进程，而不是前台运行
daemonize = true

# 30行 进程ID文件，用于进程锁，防止启动多个进程
pid_file = "/var/run/janus.pid"

# 50行 管理员密码，这个密码是web前端调用janus管理员接口时需要的参数
admin_secret = "admin"

# 202、203行 证书路径
cert_pem = "/xxx/xxx/xxx.pem"
cert_key = "/xxx/xxx/xxx.key"

# 284行、285行 STUN服务器信息
# IP、域名和端口参考上面步骤coturn服务器的配置信息来填写
stun_server = "192.168.23.176"
stun_port = 3478

# 286行 是否启用双向STUN
# 默认情况下，客户端在NAT内网，数据流通过STUN传给Janus，而服务器处在公网，可以直接发送SDP数据不经过STUN，如果需要服务器也走STUN，则启动此项
# 不配置（注释着）时，默认为false
#full_trickle = false

# 329-333行 TURN服务器信息
# IP、域名和端口参考上面步骤coturn服务器的配置信息来填写
# 用户名和密码不配置就是匿名登录使用
turn_server = "192.168.23.176"
turn_port = 3478
turn_type = "udp"
# turn_user = "username"
# turn_pwd = "password"
```

### 2. 修改 http 和 websocket 的配置文件

两份配置文件内容分为四大块

- general 普通用户
- admin 管理员用户
- cors 跨域安全
- certificates 证书

**两者除了端口号，其他配置信息大致相同**

#### general 普通用户

```
baseh_path="/janus"

http = true        ws = true
port = 8088        ws_port = 8188

https = true        wss = true
secure_port = 8089        wss_port = 8989
```

#### admin 管理员用户

```
admin_base_path = "/admin"

admin_http = true        admin_ws = true
admin_port = 7088        admin_ws_port = 7188

admin_https = true        admin_wss = true
admin_secure_port = 7889        admin_wss_port = 7989
```

#### cors 跨域安全

```
allow_origin = "*"
```

#### certificates 证书

```
cert_pem = "/xxx/xxx/xxx.pem"
cert_key = "/xxx/xxx/xxx.key"
cert_pwd = "abc123"
```

## 七、配置外网防火墙

将一些端口对外开放，必需的有：

- 3478
  用于 NAT 穿透服务器（STUN、TURN）
- 8088 8089 7088 7889
  Jauns 服务器对客户端开放的 HTTP(S)端口
- 8188 8989 7188 7989
  Janus 服务器对客户端开放的 WebSocket(s)端口

## 八、启动 Janus 守护进程

```bash
janus -b
```

因为官方没有提供停止进程的命令或者参数，所以如果需要停止 Janus，有以下两种方法参考

1. 法一（快捷简单）：读取 Janus PID 文件并杀死（注意是反引号 ` 不是单引号 ' ）

   ```bash
   kill `cat /var/run/janus.pid`
   ```

2. 法二（古法经典）： 从 Linux 进程列表里筛选出 Janus 进程并杀死
   ```bash
   ps -ef | grep janus
   kill [janus-pid]
   ```

## 九、外网测试

### 1. 端口测试

可以在另外物理地址的设备上试探端口

```bash
# 测试 STUN / TURN 服务器
curl [正式服务器域名或者IP地址]:3478
# 测试 Janus HTTP 服务器
curl [正式服务器域名或者IP地址]:8088
# 测试 Janus WebSocket 服务器
curl [正式服务器域名或者IP地址]:8188
```

不用管返回信息是什么 301、403 等状态码报错信息，至少服务器已经通了

基础设施的安装配置调试过程到此结束

后续工作由前后端在业务层面上联调测试功能
