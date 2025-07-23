# 云服务器 CentOS 安装 Janus

```bash
yum install jansson-devel libconfig-devel \
 openssl-devel nettle-devel gnutls-devel \
 glib2-devel zlib-devel pkgconfig \
 libconfig-devel libtool autoconf automake \
 cmake3 libcurl-devel \
 doxygen graphviz

pip3 install meson
pip3 install ninja

yum info libsrtp-devel
yum remove libsrtp-devel
yum info libnice-devel
yum remove libnice-devel

cd ~ && mkdir janus-setup && cd janus-setup

wget -O sofia-sip-1.13.15.zip \
https://codeload.github.com/freeswitch/sofia-sip/zip/refs/tags/v1.13.15
unzip sofia-sip-1.13.15.zip
cd sofia-sip-1.13.15
sh autogen.sh (if building from darcs)
./configure --prefix=/usr/local
make
make install
# make uninstall
ldconfig
pkg-config --modversion sofia-sip-ua

wget -O libsrtp-2.0.0.zip \
https://codeload.github.com/cisco/libsrtp/zip/refs/tags/v2.0.0
unzip libsrtp-2.0.0.zip
cd libsrtp-2.0.0
./configure --prefix=/usr/local --enable-openssl
make
make runtest
make install

------ 到这里了

# 升级cmake到cmake3
yum install cmake3
cd /usr/bin
cp cmake cmake.v2.bak && cp ccmake ccmake.v2.bak
rm cmake && rm ccmake
ln -s cmake3 cmake && ln -s ccmake3 ccmake

wget -O libnice-0.1.18.zip \
https://codeload.github.com/libnice/libnice/zip/refs/tags/0.1.18
unzip libnice-0.1.18.zip
cd libnice-0.1.18
meson build
cd build
ninja
ninja test
ninja install
pkg-config --modversion nice

# https://github.com/sctplab/usrsctp/archive/refs/tags/0.9.5.0.zip
wget -O usrsctp-0.9.5.0.zip \
https://codeload.github.com/sctplab/usrsctp/zip/refs/tags/0.9.5.0
./bootstrap
./configure --prefix=/usr/local
make
make install
ldconfig

wget -O libmicrohttpd-0.9.77.tag.gz \
https://ftp.gnu.org/gnu/libmicrohttpd/libmicrohttpd-0.9.77.tar.gz
tar zxvf libmicrohttpd-0.9.77.tag.gz
cd libmicrohttpd-0.9.77
./configure --prefix=/usr/local \
--enable-https=yes --with-ssl
make
make install
ldconfig

wget -O libwebsockets-4.3.2.zip \
https://github.com/warmcat/libwebsockets/archive/refs/tags/v4.3.2.zip
unzip libwebsockets-4.3.2.zip
cd libwebsockets-4.3.2
mkdir build
cd build
cmake \
-DCMAKE_INSTALL_PREFIX:PATH=/usr/local \
-DCMAKE_C_FLAGS="-fpic" ..
make
make install
ldconfig
# make help
# 因为Makefile没有提供卸载指令，所以从安装记录里手动卸载
xargs rm < install_manifest.txt

wget -O janus-gateway-1.1.4.zip \
https://codeload.github.com/meetecho/janus-gateway/zip/refs/tags/v1.1.4
unzip janus-gateway-1.1.4.zip
cd janus-gateway-1.1.4
sh autogen.sh
./configure --prefix=/opt/janus \
--disable-docs
make
make install
make configs

cd /etc
mkdir cert
cd cert
# 生成证书文件
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out public.crt
# 将crt证书转换成pem格式
openssl x509 -inform PEM -in public.crt > cert.pem
# 将key证书转换成pem格式
openssl rsa -in private.key -text > key.pem

```

```bash
#Janus
#if cannot configure plugin sofia,Perhaps you should add the directory containing `sofia-sip-ua.pc' to the PKG_CONFIG_PATH environment variable,
#for example centos7 :export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/lib/pkgconfig
#if cannot load libsofia-sip-ua.so.0 , try ldconfig -v
git clone https://github.com/meetecho/janus-gateway.git && \
cd janus-gateway &&\
sh autogen.sh && \
./configure --prefix=/opt/janus --disable-rabbitmq --disable-docs --disable-libsrtp2  &&\
make && make install && make configs



参考官网
GitHub - meetecho/janus-gateway: Janus WebRTC Server
https://github.com/meetecho/janus-gateway
参考文档2
WebRTC音视频技术入门与提高-Janus - 知乎
https://zhuanlan.zhihu.com/p/217655939
参考文档3
(97条消息) centos部署janus -(CentOS 7.6安装janus v0.10.10)_no unix sockets server started, giving up_wangxudongx的博客-CSDN博客
https://blog.csdn.net/wangxudongx/article/details/115655901
(97条消息) WebRTC之搭建coturn服务遇到的问题_coturn 不同网段 未生效_死磕音视频的博客-CSDN博客
https://blog.csdn.net/qq_28880087/article/details/106960293
​


https://docs.freeswitch.org/
```
