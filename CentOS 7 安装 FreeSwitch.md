看 freeswitch 里的 Makefile，每种操作系统和包管理器，需要安装的包

<http://freeswitch.org.cn/blog/2009/11/beginners-guide/>

```bash
# src dir
cd /usr/local/src
mkdir freeswitch
cd freeswitch

# basic
yum install git gcc-c++ autoconf automake libtool python ncurses-devel zlib-devel libjpeg-devel openssl-devel e2fsprogs-devel curl-devel pcre-devel speex-devel python-devel python3-devel

# unzip
yum install unzip

# cmake3
yum install cmake3
cd /usr/bin
mv cmake cmake.bak
cp cmake3 cmake3.bak
ln -s cmake3 cmake
cd /usr/local/src/freeswitch

# Sofia-sip
wget -O sofia-sip-1.13.16.zip https://github.com/freeswitch/sofia-sip/archive/refs/tags/v1.13.16.zip
unzip sofia-sip-1.13.16.zip
cd sofia-sip-1.13.16
./autogen.sh
./configure
make
make install
# make uninstall
# /usr/local
ldconfig
cd ..

# SpanDSP
wget -O spandsp-3.0.0.zip https://github.com/freeswitch/spandsp/archive/refs/heads/master.zip
unzip spandsp-3.0.0
mv spandsp-master spandsp-3.0.0
cd spandsp-3.0.0
./autogen.sh
./configure
make
make install
# make uninstall
# /usr/local
ldconfig
cd ..

# libks for mod_verto
yum install libuuid-devel libatomic openssl-devel
wget -O libks-2.0.2.zip https://github.com/signalwire/libks/archive/refs/tags/v2.0.2.zip
unzip libks-2.0.2.zip
cd libks-2.0.2
cmake . -DCMAKE_INSTALL_PREFIX:PATH=/usr/local
make
make install
# make uninstall
# 不加DCMAKE_INSTALL_PREFIX默认/usr
ldconfig
cd ..

yum install yum-plugin-ovl centos-release-scl rpmdevtools yum-utils git
yum install devtoolset-7
# 添加到profile永久生效
source /opt/rh/devtoolset-7/enable

yum install unixODBC-devel sqlite-devel
yum install erlang
yum install yasm
yum install ldns-devel libedit-devel libvpx-devel libvpx-utils opus-devel lua-devel libsndfile-devel libtiff-devel

## ffmpeg
yum install epel-release
yum update
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
yum install ffmpeg ffmpeg-devel
ldconfig

# freeswitch
# https://developer.signalwire.com/freeswitch/FreeSWITCH-Explained/Installation/Linux/CentOS-7-and-RHEL-7_10289546/
wget -O freeswitch-1.10.10.zip https://github.com/signalwire/freeswitch/archive/refs/tags/v1.10.10.zip
unzip freeswitch-1.10.10.zip
cd freeswitch-1.10.10
./bootstrap.sh -j
vim modules.conf
#applications/mod_signalwire
# ./configure --enable-portable-binary \
#     --prefix=/usr/local --localstatedir=/var --sysconfdir=/etc \
#     --with-gnu-ld --with-python3 --with-erlang --with-openssl \
#     --enable-core-odbc-support
./configure --prefix=/usr/local --localstatedir=/var --sysconfdir=/etc \
--with-gnu-ld --with-python3 --with-erlang --with-openssl \
--enable-core-odbc-support
# 每次make出错，排查问题后，都要重新configure
make
make install
make -j cd-sounds-install
make -j cd-moh-install
cd ..
```
