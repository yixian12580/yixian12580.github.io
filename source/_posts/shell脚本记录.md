---
title: shell脚本记录
tags: 脚本
categories:
  - 技术
  - 脚本
abbrlink: 4f3e6852
date: 2022-09-15 18:13:29
---

在工作中经常会有重复性的工作，这种工作借助脚本帮助我们完成可以省心又省力，记录一下工作中用到的脚本

<!--more-->

### php

#### php5.6.10安装

```bash
#!/bin/bash
#
# Description: This script is used to install php.
#

PackageDir=/opt/tools

# 安装依赖包
yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers >/dev/null
yum -y install gd-devel libjpeg-devel wget lrzsz libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel curl-devel  >/dev/null



# 创建安装包存放路径
[ ! -d ${PackageDir} ] && mkdir -p ${PackageDir} && cd ${PackageDir}


# 安装libiconv
cd ${PackageDir}
[ -f libiconv-1.14.tar.gz ] && rm -rf libiconv-1.14.tar.gz
[ -d libiconv-1.14 ] && rm -rf libiconv-1.14
wget -q http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
tar -zxf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure --prefix=/usr/local >/dev/null 2>/dev/null 1>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libiconv successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libiconv failed.\033[0m"
else 
     echo -e "\033[1;31mSome errors occured during configure libiconv.\033[0m"
     exit 9
fi

	 
# 安装libmcrypt
cd ${PackageDir}
[ -f libmcrypt-2.5.8.tar.gz ] && rm -rf libmcrypt-2.5.8.tar.gz
[ -d libmcrypt-2.5.8 ] && rm -rf libmcrypt-2.5.8
wget -q http://nchc.dl.sourceforge.net/project/mcrypt/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
tar -zxf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libmcrypt successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libmcrypt failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libmcrypt.\033[0m"
	 exit 8
fi
# 安装libltdl
ldconfig 
cd libltdl 
./configure --enable-ltdl-install >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libltdl successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libltdl failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libltdl.\033[0m"
	 exit 7
fi


# 安装mhash
cd ${PackageDir}
[ -f mhash-0.9.9.9.tar.gz ] && rm -rf mhash-0.9.9.9.tar.gz
[ -d mhash-0.9.9.9 ] && rm -rf mhash-0.9.9.9
wget -q http://nchc.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz
tar -zxf mhash-0.9.9.9.tar.gz
cd mhash-0.9.9.9
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall mhash successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install mhash failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure mhash.\033[0m"
	 exit 6
fi



# 创建lib库软连接
ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la
ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so
ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4
ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8
ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a
ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la
ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so
ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2
ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1
ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config


# 安装mcrypt
cd ${PackageDir}
[ -f mcrypt-2.6.8.tar.gz ] && rm -rf mcrypt-2.6.8.tar.gz
[ -d mcrypt-2.6.8 ] && rm -rf mcrypt-2.6.8
wget -q http://ncu.dl.sourceforge.net/project/mcrypt/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
tar -zxf mcrypt-2.6.8.tar.gz
cd mcrypt-2.6.8
ldconfig
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
     [ $? -eq 0 ] && echo -e "\033[1;32mInstall mcrypt successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install mcrypt failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure mcrypt.\033[0m"
	 exit 5
fi


# 创建php用户
useradd -u 607 -s /sbin/nologin www
[ $? -eq 0 ] && echo -e  "\033[1;32mPHP user www added.\033[0m" || echo -e "\033[1;31mAdd php user failed.\033[0m"


# 安装PHP
PVERSION=5.6.10
cd ${PackageDir}
[ -f php-${PVERSION}.tar.gz ] && rm -rf php-${PVERSION}.tar.gz
[ -d php-${PVERSION} ] && rm -rf php-${PVERSION}
wget -q http://cn2.php.net/distributions/php-${PVERSION}.tar.gz
tar -zxf php-${PVERSION}.tar.gz 
cd php-${PVERSION}
./configure --prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-libxml-dir \
--enable-xml \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-bcmath \
--enable-mbstring \
--enable-gd-native-ttf \
--enable-sockets \
--enable-mysqlnd \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-zip \
--enable-inline-optimization \
--with-gd \
--with-bz2 \
--with-zlib \
--with-mcrypt \
--with-mhash \
--with-openssl \
--with-xmlrpc \
--with-iconv-dir \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--without-pear \
--disable-ipv6 \
--disable-pdo \
--with-gettext \
--disable-debug \
--without-pdo-sqlite \
--disable-rpath \
--enable-shmop \
--enable-sysvsem \
--with-curl \
--enable-mbregex \
--enable-pcntl \
--enable-soap \
--enable-sigchild \
--enable-pdo  >/dev/null 2>/dev/null

if [ $? -eq 0 ]; then
     make ZEND_EXTRA_LIBS='-liconv' >/dev/null 2>/dev/null 1>/dev/null
	 if [ $? -eq 0 ]; then
	     make install >/dev/null 2>/dev/null 1>/dev/null
		 [ $? -eq 0 ] && echo -e "\033[1;32mInstall php${PVERSION} successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install php${PVERSION} failed.\033[0m"
	 else
	     echo -e "\033[1;31mSome errors occured during make php${PVERSION}.\033[0m"
		 exit 4
	 fi
else
     echo -e "\033[1;31mSome errors occured during configure php${PVERSION}.\033[0m"
	 exit 3
fi
	 

# 添加php到环境变量
echo -e 'PATH=$PATH:/usr/local/php/bin:/usr/local/php/sbin' >> /etc/profile
source /etc/profile


# 创建php-fpm配置文件
cd /usr/local/php/etc/
[  -e php-fpm.conf ] && mv php-fpm.conf php-fpm.conf.bak$(date +%F)
cp php-fpm.conf.default php-fpm.conf

# 生成php.ini文件
[ -e /etc/php.ini ] && mv /etc/php.ini /etc/php.ini.bak$(date +%F)
cp /opt/tools/php-5.6.10/php.ini-production /etc/php.ini


# 配置php-fpm启动脚本并添加开机启动
[ -e /etc/init.d/php-fpm ] && mv /etc/init.d/php-fpm /etc/init.d/php-fpm.bak$(date +%F)
cp /opt/tools/php-5.6.10/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm
chkconfig --add php-fpm
chkconfig --list|grep php-fpm >/dev/null
if [ $? -eq 0 ]; then
     echo -e "\033[1;32mAdd php-fpm into chkconfig list successful.\033[0m" || echo -e "\033[1;31mAdd php-fpm into chkconfig list failed.\033[0m"
     chkconfig php-fpm on 
     chkconfig --list|grep 3:on|grep php >/dev/null
	 if [ $? -eq 0 ];then
	     echo -e "\033[1;32mChange php-fpm status to on in chkconfig list successful.\033[0m"
	 else
	     echo -e "\033[1;31mChange php-fpm status to on in chkconfig list failed.\033[0m"
	 fi
else
     echo -e "\033[1;31mAdd php-fpm into chkconfig list failed.\033[0m"
fi


# 启动php-fpm
/etc/init.d/php-fpm start >/dev/null
[ $? -eq 0 ] && echo -e "\033[1;32mStarting php-fpm successful.\033[0m" || echo -e "\033[1;31mStarting php-fpm failed.\033[0m"

.  /etc/profile
source /etc/profile


# 安装libmemcached依赖
cd ${PackageDir}
wget -q https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
tar -zxf libmemcached-1.0.18.tar.gz 
cd libmemcached-1.0.18
./configure --prefix=/usr/local/libmemcached --with-memcached >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libmemcached package successful.\033[0m" || echo -e "\033[1;31mInstall libmemcached package failed.\033[0m"
else
	 echo -e "\033[1;31mSome errors occured during configure libmemcached.\033[0m"
	 exit 11
fi

# 安装memcached包
cd ${PackageDir}
wget -q https://pecl.php.net/get/memcached-2.2.0.tgz
tar -zxf memcached-2.2.0.tgz 
cd memcached-2.2.0
/usr/local/php/bin/phpize 
./configure --enable-memcached --with-php-config=/usr/local/php/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached  >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then 
     make && make install >/dev/null 2>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall memcached package successful.\033[0m" || echo -e "\033[1;31mInstall memcached package failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure memcached.\033[0m"
	 exit 12
fi

# 添加memcached扩展
[ -e /usr/local/php/etc/php.ini ] && mv /usr/local/php/etc/php.ini /usr/local/php/etc/php.ini.bak 
cp /etc/php.ini /usr/local/php/etc/
echo 'extension=memcached.so' >> /usr/local/php/etc/php.ini 
/usr/local/php/bin/php -m |grep memcached >/dev/null
[ $? -eq 0 ] && echo -e "\033[1;32mAdd memcached extension to php successful.\033[0m" || echo -e "\033[1;31mAdd memcached extension to php failed.\033[0m"

# 安装phpredis包
cd ${PackageDir}
wget -q https://github.com/phpredis/phpredis/archive/2.2.4.tar.gz
tar -zxf 2.2.4.tar.gz
cd phpredis-2.2.4/
/usr/local/php/bin/phpize 
./configure --with-php-config=/usr/local/php/bin/php-config >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null
     [ $? -eq 0 ] && echo -e "\033[1;32mInstall phpredis package successful.\033[0m" || echo -e "\033[1;31mInstall phpredis package failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure phpredis.\033[0m"
	 exit 13
fi

# 添加redis扩展
echo 'extension=redis.so' >> /usr/local/php/etc/php.ini
/usr/local/php/bin/php -m |grep redis >/dev/null
[ $? -eq 0 ] && echo -e "\033[1;32mAdd redis extension to php successful.\033[0m" || echo -e "\033[1;31mAdd redis extension to php failed.\033[0m"


chown -R www. /usr/local/php/
. /etc/profile
```

#### php7.3.4安装

```bash
#!/bin/bash
#
# Description: This script is used to install php.
#

source /etc/profile
PackageDir=/opt/tools

# 安装依赖包
yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers >/dev/null
yum -y install gd-devel libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel curl-devel  >/dev/null



# 创建安装包存放路径
[ ! -d ${PackageDir} ] && mkdir -p ${PackageDir} && cd ${PackageDir}


# 安装libiconv
cd ${PackageDir}
[ -f libiconv-1.14.tar.gz ] && rm -rf libiconv-1.14.tar.gz
[ -d libiconv-1.14 ] && rm -rf libiconv-1.14
wget -q http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
tar -zxf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure --prefix=/usr/local >/dev/null 2>/dev/null 1>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libiconv successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libiconv failed.\033[0m"
else 
     echo -e "\033[1;31mSome errors occured during configure libiconv.\033[0m"
     exit 9
fi

	 
# 安装libmcrypt
cd ${PackageDir}
[ -f libmcrypt-2.5.8.tar.gz ] && rm -rf libmcrypt-2.5.8.tar.gz
[ -d libmcrypt-2.5.8 ] && rm -rf libmcrypt-2.5.8
wget -q http://nchc.dl.sourceforge.net/project/mcrypt/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
tar -zxf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libmcrypt successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libmcrypt failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libmcrypt.\033[0m"
	 exit 8
fi
# 安装libltdl
/sbin/ldconfig 
cd libltdl 
./configure --enable-ltdl-install >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libltdl successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libltdl failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libltdl.\033[0m"
	 exit 7
fi


# 安装mhash
cd ${PackageDir}
[ -f mhash-0.9.9.9.tar.gz ] && rm -rf mhash-0.9.9.9.tar.gz
[ -d mhash-0.9.9.9 ] && rm -rf mhash-0.9.9.9
wget -q http://nchc.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz
tar -zxf mhash-0.9.9.9.tar.gz
cd mhash-0.9.9.9
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall mhash successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install mhash failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure mhash.\033[0m"
	 exit 6
fi



# 创建lib库软连接
ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la
ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so
ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4
ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8
ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a
ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la
ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so
ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2
ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1
ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config
/sbin/ldconfig


# 安装mcrypt
cd ${PackageDir}
[ -f mcrypt-2.6.8.tar.gz ] && rm -rf mcrypt-2.6.8.tar.gz
[ -d mcrypt-2.6.8 ] && rm -rf mcrypt-2.6.8
wget -q https://nchc.dl.sourceforge.net/project/mcrypt/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
tar -zxf mcrypt-2.6.8.tar.gz
cd mcrypt-2.6.8
/sbin/ldconfig
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
     [ $? -eq 0 ] && echo -e "\033[1;32mInstall mcrypt successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install mcrypt failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure mcrypt.\033[0m"
	 exit 5
fi

# 安装libzip1.2
cd ${PackageDir}
[ -f libzip-1.2.0.tar.gz ] && rm -rf libzip-1.2.0.tar.gz
[ -d libzip-1.2.0 ] && rm -rf libzip-1.2.0
wget -q https://nih.at/libzip/libzip-1.2.0.tar.gz
tar -zxf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
     [ $? -eq 0 ] && echo -e "\033[1;32mInstall libzip successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libzip failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libzip.\033[0m"
     exit 15
fi

# 指定zipconf声明文件路径
cp /usr/local/lib/libzip/include/zipconf.h /usr/local/include/zipconf.h

# 添加搜索路径到配置文件
echo '/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64' >> /etc/ld.so.conf
# 更新配置
/sbin/ldconfig -v



# 创建php用户
useradd -u 607 -s /sbin/nologin www
[ $? -eq 0 ] && echo -e  "\033[1;32mPHP user www added.\033[0m" || echo -e "\033[1;31mAdd php user failed.\033[0m"


# 安装PHP
PVERSION=7.3.4
cd ${PackageDir}
[ -f php-${PVERSION}.tar.gz ] && rm -rf php-${PVERSION}.tar.gz
[ -d php-${PVERSION} ] && rm -rf php-${PVERSION}
#wget -q http://cn2.php.net/distributions/php-${PVERSION}.tar.gz
wget -q http://ftp.ntu.edu.tw/php/distributions/php-7.3.4.tar.gz
tar -zxf php-${PVERSION}.tar.gz 
cd php-${PVERSION}
./configure --prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-libxml-dir \
--enable-xml \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-bcmath \
--enable-mbstring \
--enable-sockets \
--enable-mysqlnd \
--enable-opcache \
--enable-session \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-mysqlnd-compression-support \
--enable-inline-optimization \
--with-gd \
--with-bz2 \
--with-zlib \
--enable-zip \
--with-mhash \
--with-openssl \
--with-xmlrpc \
--with-iconv-dir \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--without-pear \
--disable-phar \
--with-pcre-regex \
--disable-ipv6 \
--with-gettext \
--disable-debug \
--without-pdo-sqlite \
--disable-rpath \
--enable-shmop \
--enable-sysvsem \
--with-curl \
--enable-mbregex \
--enable-pcntl \
--enable-soap \
--enable-sigchild \
--enable-pdo   

if [ $? -eq 0 ]; then
     make ZEND_EXTRA_LIBS='-liconv' 
	 if [ $? -eq 0 ]; then
	     make install 
		 [ $? -eq 0 ] && echo -e "\033[1;32mInstall php${PVERSION} successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install php${PVERSION} failed.\033[0m"
	 else
	     echo -e "\033[1;31mSome errors occured during make php${PVERSION}.\033[0m"
		 exit 4
	 fi
else
     echo -e "\033[1;31mSome errors occured during configure php${PVERSION}.\033[0m"
	 exit 3
fi
	 

# 添加php到环境变量
echo -e 'PATH=$PATH:/usr/local/php/bin:/usr/local/php/sbin' >> /etc/profile
source /etc/profile


# 创建php-fpm配置文件
cd /usr/local/php/etc/
[  -e php-fpm.conf ] && mv php-fpm.conf php-fpm.conf.bak$(date +%F)
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf

# 生成php.ini文件
[ -e /etc/php.ini ] && mv /etc/php.ini /etc/php.ini.bak$(date +%F)
cp ${PackageDir}/php-${PVERSION}/php.ini-production /etc/php.ini


# 配置php-fpm启动脚本并添加开机启动
[ -e /etc/init.d/php-fpm ] && mv /etc/init.d/php-fpm /etc/init.d/php-fpm.bak$(date +%F)
cp ${PackageDir}/php-${PVERSION}/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm
chkconfig --add php-fpm
chkconfig --list|grep php-fpm >/dev/null
if [ $? -eq 0 ]; then
     echo -e "\033[1;32mAdd php-fpm into chkconfig list successful.\033[0m" || echo -e "\033[1;31mAdd php-fpm into chkconfig list failed.\033[0m"
     chkconfig php-fpm on 
     chkconfig --list|grep 3:on|grep php >/dev/null
	 if [ $? -eq 0 ];then
	     echo -e "\033[1;32mChange php-fpm status to on in chkconfig list successful.\033[0m"
	 else
	     echo -e "\033[1;31mChange php-fpm status to on in chkconfig list failed.\033[0m"
	 fi
else
     echo -e "\033[1;31mAdd php-fpm into chkconfig list failed.\033[0m"
fi

```

### Nginx

#### Nginx1.6.3安装

```
#!/bin/bash
#
# Description: This script is used to install php.
#

source /etc/profile
PackageDir=/opt/tools

# 安装依赖包
yum -y install gcc gcc-c++ autoconf libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers >/dev/null
yum -y install gd-devel libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel curl-devel  >/dev/null



# 创建安装包存放路径
[ ! -d ${PackageDir} ] && mkdir -p ${PackageDir} && cd ${PackageDir}


# 安装libiconv
cd ${PackageDir}
[ -f libiconv-1.14.tar.gz ] && rm -rf libiconv-1.14.tar.gz
[ -d libiconv-1.14 ] && rm -rf libiconv-1.14
wget -q http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
tar -zxf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure --prefix=/usr/local >/dev/null 2>/dev/null 1>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libiconv successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libiconv failed.\033[0m"
else 
     echo -e "\033[1;31mSome errors occured during configure libiconv.\033[0m"
     exit 9
fi

	 
# 安装libmcrypt
cd ${PackageDir}
[ -f libmcrypt-2.5.8.tar.gz ] && rm -rf libmcrypt-2.5.8.tar.gz
[ -d libmcrypt-2.5.8 ] && rm -rf libmcrypt-2.5.8
wget -q http://nchc.dl.sourceforge.net/project/mcrypt/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
tar -zxf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libmcrypt successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libmcrypt failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libmcrypt.\033[0m"
	 exit 8
fi
# 安装libltdl
/sbin/ldconfig 
cd libltdl 
./configure --enable-ltdl-install >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall libltdl successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libltdl failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libltdl.\033[0m"
	 exit 7
fi


# 安装mhash
cd ${PackageDir}
[ -f mhash-0.9.9.9.tar.gz ] && rm -rf mhash-0.9.9.9.tar.gz
[ -d mhash-0.9.9.9 ] && rm -rf mhash-0.9.9.9
wget -q http://nchc.dl.sourceforge.net/project/mhash/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz
tar -zxf mhash-0.9.9.9.tar.gz
cd mhash-0.9.9.9
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
	 [ $? -eq 0 ] && echo -e "\033[1;32mInstall mhash successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install mhash failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure mhash.\033[0m"
	 exit 6
fi



# 创建lib库软连接
ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la
ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so
ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4
ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8
ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a
ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la
ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so
ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2
ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1
ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config
/sbin/ldconfig


# 安装mcrypt
cd ${PackageDir}
[ -f mcrypt-2.6.8.tar.gz ] && rm -rf mcrypt-2.6.8.tar.gz
[ -d mcrypt-2.6.8 ] && rm -rf mcrypt-2.6.8
wget -q https://nchc.dl.sourceforge.net/project/mcrypt/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
tar -zxf mcrypt-2.6.8.tar.gz
cd mcrypt-2.6.8
/sbin/ldconfig
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
     [ $? -eq 0 ] && echo -e "\033[1;32mInstall mcrypt successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install mcrypt failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure mcrypt.\033[0m"
	 exit 5
fi

# 安装libzip1.2
cd ${PackageDir}
[ -f libzip-1.2.0.tar.gz ] && rm -rf libzip-1.2.0.tar.gz
[ -d libzip-1.2.0 ] && rm -rf libzip-1.2.0
wget -q https://nih.at/libzip/libzip-1.2.0.tar.gz
tar -zxf libzip-1.2.0.tar.gz
cd libzip-1.2.0
./configure >/dev/null 2>/dev/null
if [ $? -eq 0 ]; then
     make && make install >/dev/null 2>/dev/null 1>/dev/null
     [ $? -eq 0 ] && echo -e "\033[1;32mInstall libzip successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install libzip failed.\033[0m"
else
     echo -e "\033[1;31mSome errors occured during configure libzip.\033[0m"
     exit 15
fi

# 指定zipconf声明文件路径
cp /usr/local/lib/libzip/include/zipconf.h /usr/local/include/zipconf.h

# 添加搜索路径到配置文件
echo '/usr/local/lib64
/usr/local/lib
/usr/lib
/usr/lib64' >> /etc/ld.so.conf
# 更新配置
/sbin/ldconfig -v



# 创建php用户
useradd -u 607 -s /sbin/nologin www
[ $? -eq 0 ] && echo -e  "\033[1;32mPHP user www added.\033[0m" || echo -e "\033[1;31mAdd php user failed.\033[0m"


# 安装PHP
PVERSION=7.3.4
cd ${PackageDir}
[ -f php-${PVERSION}.tar.gz ] && rm -rf php-${PVERSION}.tar.gz
[ -d php-${PVERSION} ] && rm -rf php-${PVERSION}
#wget -q http://cn2.php.net/distributions/php-${PVERSION}.tar.gz
wget -q http://ftp.ntu.edu.tw/php/distributions/php-7.3.4.tar.gz
tar -zxf php-${PVERSION}.tar.gz 
cd php-${PVERSION}
./configure --prefix=/usr/local/php \
--with-config-file-path=/usr/local/php/etc \
--with-libxml-dir \
--enable-xml \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-bcmath \
--enable-mbstring \
--enable-sockets \
--enable-mysqlnd \
--enable-opcache \
--enable-session \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-mysqlnd-compression-support \
--enable-inline-optimization \
--with-gd \
--with-bz2 \
--with-zlib \
--enable-zip \
--with-mhash \
--with-openssl \
--with-xmlrpc \
--with-iconv-dir \
--with-freetype-dir \
--with-jpeg-dir \
--with-png-dir \
--without-pear \
--disable-phar \
--with-pcre-regex \
--disable-ipv6 \
--with-gettext \
--disable-debug \
--without-pdo-sqlite \
--disable-rpath \
--enable-shmop \
--enable-sysvsem \
--with-curl \
--enable-mbregex \
--enable-pcntl \
--enable-soap \
--enable-sigchild \
--enable-pdo   

if [ $? -eq 0 ]; then
     make ZEND_EXTRA_LIBS='-liconv' 
	 if [ $? -eq 0 ]; then
	     make install 
		 [ $? -eq 0 ] && echo -e "\033[1;32mInstall php${PVERSION} successful.\033[0m" || echo -e "\033[1;31mSome errors occured, install php${PVERSION} failed.\033[0m"
	 else
	     echo -e "\033[1;31mSome errors occured during make php${PVERSION}.\033[0m"
		 exit 4
	 fi
else
     echo -e "\033[1;31mSome errors occured during configure php${PVERSION}.\033[0m"
	 exit 3
fi
	 

# 添加php到环境变量
echo -e 'PATH=$PATH:/usr/local/php/bin:/usr/local/php/sbin' >> /etc/profile
source /etc/profile


# 创建php-fpm配置文件
cd /usr/local/php/etc/
[  -e php-fpm.conf ] && mv php-fpm.conf php-fpm.conf.bak$(date +%F)
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf

# 生成php.ini文件
[ -e /etc/php.ini ] && mv /etc/php.ini /etc/php.ini.bak$(date +%F)
cp ${PackageDir}/php-${PVERSION}/php.ini-production /etc/php.ini


# 配置php-fpm启动脚本并添加开机启动
[ -e /etc/init.d/php-fpm ] && mv /etc/init.d/php-fpm /etc/init.d/php-fpm.bak$(date +%F)
cp ${PackageDir}/php-${PVERSION}/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
chmod 755 /etc/init.d/php-fpm
chkconfig --add php-fpm
chkconfig --list|grep php-fpm >/dev/null
if [ $? -eq 0 ]; then
     echo -e "\033[1;32mAdd php-fpm into chkconfig list successful.\033[0m" || echo -e "\033[1;31mAdd php-fpm into chkconfig list failed.\033[0m"
     chkconfig php-fpm on 
     chkconfig --list|grep 3:on|grep php >/dev/null
	 if [ $? -eq 0 ];then
	     echo -e "\033[1;32mChange php-fpm status to on in chkconfig list successful.\033[0m"
	 else
	     echo -e "\033[1;31mChange php-fpm status to on in chkconfig list failed.\033[0m"
	 fi
else
     echo -e "\033[1;31mAdd php-fpm into chkconfig list failed.\033[0m"
fi

```

#### nginx日志切割

```bash
#!/bin/bash
############################################
# Description: This script is used to cut  #
#              nginx logs.                 #
############################################

LogsPath=/usr/local/nginx/logs
BakDir=/data/backup/nginx_logs
PidFile=/usr/local/nginx/logs/nginx.pid
Yesterday=`date -d 'yesterday' +%F`
OLDDATE=`date -d '15 days ago' +%F`


#[ ! -d ${BakDir} ] && mkdir -p ${BakDir}

[ ! -d ${BakDir}/${Yesterday} ] && mkdir -p ${BakDir}/${Yesterday} || rm -rf ${BakDir}/${Yesterday}/* 
if [ ! -d ${LogsPath} ];then
     echo "Cannot find nginx log path."
     exit 1
else 
     cd ${LogsPath}
	     for log in `ls *.log`;do
		 mv ${LogsPath}/${log} ${BakDir}/${Yesterday}/
	         kill -USR1 `cat $PidFile`
	         sleep 1
	         cd ${BakDir}/${Yesterday}/
                 tar -zcf ${log}.tar.gz $log
                 rm -rf $log
             done
     chown -R nginx.nginx ${LogsPath}

     # 检查并删除过期的日志文件
     cd ${BakDir} && rm -rf ${OLDDATE}
fi

```



### 检查域名到期时间

#### 脚本1（python）

```python
# -*- coding:utf-8 -*-

"""
Notice: This script is used to check SSL certificate expire date. Need Python3 env and requests module.
"""

import re
import time
import requests
import json
from datetime import datetime
import traceback

"""
Notice: This script is used to check domain expire date. 
Env: Python3 + requests module
"""


# 定义用到的whois查询站点URL
whois_site = 'http://whoissoft.com/'

# 定义要检查的域名列表
domains = ['1.com', '2.cn', ]

# 定义钉钉告警组
robot_token = {
    'ops': 'f0f58a2d65412e2c34da2c579627c571212b6************'
}
left_days = 0

# 定义一个钉钉告警函数
def alert_by_dingding(grpname, webhook_title, alert_message):
    if grpname in robot_token:
        try:
            token = robot_token[grpname]
            webhook_url = 'https://oapi.dingtalk.com/robot/send?access_token=%s' % token

            webhook_header = {
                "Content-Type": "application/json",
                "charset": "utf-8"
            }
            webhook_message = {
                "msgtype": "markdown",
                "markdown": {
                    "title": webhook_title,
                    "text": alert_message
                }
            }

            sendData = json.dumps(webhook_message, indent=1)
            requests.post(url=webhook_url, headers=webhook_header, data=sendData)
        except Exception as e:
            traceback.print_exc(file=open('/tmp/alert_by_dingding.log', 'w+'))


# 定义一个用来获取域名信息的函数
def get_domain_info(domain):
    req_url = whois_site + domain
    headers = {'user-agent': 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) Gecko/2008071615 Fedora/3.0.1-1.fc9 '
                             'Firefox/3.0.1'}
    r = requests.get(req_url, headers=headers, timeout=30)
    if r.status_code == requests.codes.ok:
        content = r.text
        try:
            # 从返回内容中获取域名到期时间
            info = re.search('Registry Expiry Date: (.*?)\n', content)
            exp_date = re.split('<br />', info.group(1))[0]
            # 将时间字符串转换为时间数组，进而计算出剩余天数
            expire_date = datetime.strptime(exp_date, '%Y-%m-%dT%H:%M:%SZ')
        except AttributeError:
            info = re.search('Expiration Time: (.*?)\n', content)
            exp_date = re.split('<br />', info.group(1))[0]
            # 将时间字符串转换为时间数组，进而计算出剩余天数
            expire_date = datetime.strptime(exp_date, '%Y-%m-%d %H:%M:%S')
        except Exception as e:
            print("Error message: %s" % (str(e)))
        finally:
            left_days = (expire_date - datetime.now()).days
            expire_day = datetime.strftime(expire_date, '%Y-%m-%d')
        

        if left_days <= 90:
            try:
                alert_title = "Domain Expiration!!!"
                alert_content = '<font size="12" color="#ff3300">【警告】</font>域名即将过期\n' \
                                '****\n' \
                                '域名：%s  \n' \
                                '到期日期：%s  \n' \
                                '剩余天数：%s' % (domain, expire_day, left_days)
                alert_by_dingding('ops', alert_title, alert_content)
            except Exception as e:
                traceback.print_exc(file=open('/tmp/alert_by_dingding.log', 'w+'))

    # 睡眠1s
    time.sleep(1)


if __name__ == '__main__':
    if len(domains) > 0:
        for domain in domains:
            get_domain_info(domain)
    else:
        print("Please check domain list first.")

```

#### 脚本2（python）

```python
# _*_ coding:utf-8 _*_

import requests
import re
import datetime
import json
import traceback


whois_site = 'http://whois.chinaz.com/'
#domains = ['2.com', '2.cn']
domains = ['jingyu.top', 'dreame.tw', 'stary.ltd']
robot_token = {
    'ops': 'f0f58a2d65412e2c34da2c579627c571212b6*********************'
}

# 获取当前日期
today = datetime.date.today()
# 将当前日期转换为datetime类型
tdate = datetime.datetime.strptime(str(today), '%Y-%m-%d')


def alert_by_dingding(grpname, webhook_title, alert_message):
    if grpname in robot_token:
        try:
            token = robot_token[grpname]
            webhook_url = 'https://oapi.dingtalk.com/robot/send?access_token=%s' % token

            webhook_header = {
                "Content-Type": "application/json",
                "charset": "utf-8"
            }
            webhook_message = {
                "msgtype": "markdown",
                "markdown": {
                    "title": webhook_title,
                    "text": alert_message
                }
            }

            sendData = json.dumps(webhook_message, indent=1)
            requests.post(url=webhook_url, headers=webhook_header, data=sendData)
        except Exception as e:
            traceback.print_exc(file=open('/tmp/alert_by_dingding.log', 'w+'))


def get_domain_expiration(domain):
    req_url = whois_site + domain
    headers = {'user-agent': 'Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.1) Gecko/2008071615 Fedora/3.0.1-1.fc9 '
                             'Firefox/3.0.1'}
    r = requests.get(req_url, headers=headers, timeout=30)
    if r.status_code == requests.codes.ok:
        content = r.text
        info = re.search('<div class="fl WhLeList-left">过期时间</div><div class="fr WhLeList-right"><span>(\w)+</span>'
                         , content).group()
        res = re.split('<span>|</span>', info)[1]
        res2 = re.sub('(\D)', '-', res).strip('-')

        # 将到期日期转换为datetime类型
        ddate = datetime.datetime.strptime(res2, '%Y-%m-%d')
        # 获取剩余天数
        leftdays = (ddate - tdate).days

        if leftdays <= 90:
            try:
                alert_title = "Domain Expiration!!!"
                alert_content = '<font size="12" color="#ff3300">【警告】</font>域名即将过期\n' \
                                '****\n' \
                                '域名：%s  \n' \
                                '到期日期：%s  \n' \
                                '剩余天数：%s' % (domain, ddate, leftdays)
                alert_by_dingding('ops', alert_title, alert_content)
            except Exception as e:
                print(alert_content)


if __name__ == '__main__':
    for domain in domains:
        get_domain_expiration(domain)

```

### 检查ssl证书到期时间（python）

```python
# -*- coding:utf-8 -*-

"""
Notice: This script is used to check SSL certificate expire date. Need Python3 env and requests module.
"""

import re
import time
import requests
import json
from datetime import datetime
import subprocess
import traceback


# 定义要检查的域名列表
domains = ['1.com', '2.cn']
# 定义钉钉告警组
robot_token = {
    'ops': 'f0f58a2d65412e2c34da2c579627c571212b6bced***********************'
}


# 定义一个钉钉告警函数
def alert_by_dingding(grpname, webhook_title, alert_message):
    if grpname in robot_token:
        try:
            token = robot_token[grpname]
            webhook_url = 'https://oapi.dingtalk.com/robot/send?access_token=%s' % token

            webhook_header = {
                "Content-Type": "application/json",
                "charset": "utf-8"
            }
            webhook_message = {
                "msgtype": "markdown",
                "markdown": {
                    "title": webhook_title,
                    "text": alert_message
                }
            }

            sendData = json.dumps(webhook_message, indent=1)
            requests.post(url=webhook_url, headers=webhook_header, data=sendData)
        except Exception as e:
            traceback.print_exc(file=open('/tmp/alert_by_dingding.log', 'w+'))


# 定义一个用来获取ssl证书信息的函数
def get_ssl_info(domain):
    cmd = 'curl -lvs https://{}/'.format(domain)
    sslinfo = subprocess.getstatusoutput(cmd)[1]
    
    try:
        # 使用正则表达获取证书过期时间
        m = re.search('subject:(.*?)\n.*?start date:(.*?)\n.*?expire date:(.*?)\n.*?common name:(.*?)\n.*?issuer:(.*?)\n',
                      sslinfo)
        start_date = m.group(2)
        expire_date = m.group(3)
        cert_name = m.group(4)
    except Exception as e:
        print("Error message: %s", str(e))
    else:
        # time字符串转换为时间数组，跟当前的时间相减得出证书剩余有效天数
        expire_date_format = datetime.strptime(expire_date, " %b %d %H:%M:%S %Y GMT")
        # expire_day仅用来保存过期日期（精确到天）,为了钉钉告警格式美观而设置
        expire_day = datetime.strftime(expire_date_format, '%Y-%m-%d')
        left_days = (expire_date_format - datetime.now()).days

    if left_days <= 90:
        try:
            alert_title = "Warning"
            alert_content = '<font size="12" color="#ff3300">【警告】</font>SSL证书即将过期\n' \
                            '****\n' \
                            '证书：%s  \n' \
                            '证书到期日：%s  \n' \
                            '剩余天数：%s' % (cert_name, expire_day, left_days)
            alert_by_dingding('ops', alert_title, alert_content)
        except Exception as e:
            #print(alert_content)
            print("Error message: %s" % str(e))

    # 睡眠1s
    time.sleep(1)


if __name__ == '__main__':
    if len(domains) >0:
        for domain in domains:
            get_ssl_info(domain)
    else:
        print("Please check domain list first.")

```

### MySQL备份

```bash
#!/bin/bash
#

MYSQL_ROOT_DIR=/usr/local/mysql
MYSQL_EXEC=$MYSQL_ROOT_DIR/bin/mysql
MYSQLDUMP_EXEC=$MYSQL_ROOT_DIR/bin/mysqldump 
MYSQL_USER=dbbackup
MYSQL_PASSWD=backup@159
DBNAMES=`${MYSQL_EXEC} -h 127.0.0.1 -u${MYSQL_USER} -p${MYSQL_PASSWD} -e "show databases;"|grep -Ev "^Database|information_schema|performance_schema|test"`
YESTERDAY=`date -d yesterday +%F`
OLDDATE=`date -d '7 days ago' +%F`
BAK_DIR=/storage/backup/mysql


[ ! -d $BAK_DIR ] && mkdir -p $BAK_DIR
[ -d $BAK_DIR/$YESTERDAY ] && rm -rf $BAK_DIR/$YESTERDAY
mkdir -p $BAK_DIR/$YESTERDAY


echo "MySQL database backup start on $(date +%F' '%T)"
echo "========================================="

for i in ${DBNAMES} 
   do 
       ${MYSQLDUMP_EXEC} --opt -h 127.0.0.1 -u${MYSQL_USER} -p${MYSQL_PASSWD} $i | gzip > ${BAK_DIR}/${YESTERDAY}/$i.sql.gz
	   [ $? -eq 0 ] && echo "Backup database $i successful." || echo "Backup database $i failed."
       sleep 5
done


	${MYSQLDUMP_EXEC} --opt -h 127.0.0.1 -u${MYSQL_USER} -p${MYSQL_PASSWD} --all-databases  | gzip > ${BAK_DIR}/${YESTERDAY}/alldatabase_${YESTERDAY}.sql.gz   
    [ $? -eq 0 ] && echo "Backup all databases successful." || echo "Backup all databases failed."

cd $BAK_DIR && rm -rf ${OLDDATE}

echo "========================================="
echo "MySQL database backup ended on $(date +%F' '%T)."	   
echo "#########################################"

```

### 检查应用状态

#### 检查应用在Google Play商店状态（python）

```python
# 检查谷歌play应用的上线情况

# 需要检查的进程列表
import requests
import sys
import datetime,time
import hmac
import hashlib
import base64
import urllib
import traceback
import json

app_dict = {
       "应用名1":'包名1',
       "应用名2":'包名2'
}

def get_status(appName):
    url = 'https://play.google.com/store/apps/details'
    try:
        response = requests.get(url=url,params={'id': appName})
    except Exception as e:
        print("连接失败:{0}".format(e))
        sys.exit(2)
    stdout = int(response.status_code)
    if stdout == 200:
        return True
    else:
        return False

# 定义钉钉告警群
def get_sign_timestamp():
    timestamp = str(round(time.time() * 1000))
    secret = 'SEC6eb32d8c662b538872b6dbe83c09674bb5********************'
    secret_enc = secret.encode('utf-8')
    string_to_sign = '{}\n{}'.format(timestamp, secret)
    string_to_sign_enc = string_to_sign.encode('utf-8')
    hmac_code = hmac.new(secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
    sign = urllib.parse.quote_plus(base64.b64encode(hmac_code))
    # print(timestamp,sign)
    return timestamp,sign

# 钉钉机器人的token值
robot_token = {
    "ops": 'c3765d92d7b11d4c919f304a0a500ca653cf82c02****************'
}


# 定义一个钉钉告警函数
def alert_by_dingding(grpname, webhook_title, alert_message):
    if grpname in robot_token:
        try:
            token = robot_token[grpname]
            timestamp,sign = get_sign_timestamp()
            url = 'https://oapi.dingtalk.com/robot/send?access_token={0}&timestamp={1}&sign={2}'.format(token,timestamp,sign)


            webhook_header = {
                "Content-Type": "application/json",
                "charset": "utf-8"
            }
            webhook_message = {
                "msgtype": "markdown",
                "markdown": {
                    "title": webhook_title,
                    "text": alert_message
                }
            }

            sendData = json.dumps(webhook_message, indent=1)
            requests.post(url=url, headers=webhook_header, data=sendData)
        except Exception as e:
            traceback.print_exc(e,file=open('/tmp/alert_by_dingding.log', 'w+'))

def utc2local():
    # “”“UTC时间转本地时间（+8:00)
    offset = datetime.timedelta(hours=8)
    utc_time = datetime.datetime.utcnow()
    print(offset)
    local_st = utc_time + offset
    return local_st

def main():
    for app in app_dict:
        result = get_status(app_dict[app])
        if result:
            print("{0}应用已上架".format(app))
            continue
        else:
            print("{0}应用未上架,将发送报警信息".format(app))
            date_now = utc2local().strftime("%Y-%m-%d %H:%M:%S")
            description="未在谷歌PLAY商店发现该应用，请检查"
            title = "检查应用上架情况"
            message = """
## 故障
**** 
**应用名称：** {appName}      
**应用包名：** {appKey}      
**告警描述：** {TRIGGER_DESCRIPTION}   
**告警时间：** {EVENT_DATE}   
**当前状态：** <font size="6" color="#ff3300">PROBLEM</font>
""".format(appName=app,appKey=app_dict[app],TRIGGER_DESCRIPTION=description,EVENT_DATE=date_now)
            alert_by_dingding('ops',title,message)

main()
```

#### 检查应用在Apple Store商店状态（python）

```python
# 检查ios应用的上线情况

# 需要检查的进程列表
import requests
import sys
import datetime,time
import hmac
import hashlib
import base64
import urllib
import traceback
import json
import time

app_dict = {
       "应用名1":'应用名1小写/idAPPID',
       "应用名2":'应用名2小写/idAPPID',
}

url = 'https://apps.apple.com/us/app/'

def build_uri(endpoint):
    return '/'.join([url, endpoint])

def get_status(appName):
    try:
        response = requests.get(build_uri(appName))
    except Exception as e:
        print("连接失败:{0}".format(e))
        sys.exit(2)
    stdout = int(response.status_code)
    if stdout == 200:
        return True
    else:
        return False
    time.sleep( 90 )

# 定义钉钉告警群
def get_sign_timestamp():
    timestamp = str(round(time.time() * 1000))
    secret = 'SEC6eb32d8c662b538872b6dbe83c0967************'
    secret_enc = secret.encode('utf-8')
    string_to_sign = '{}\n{}'.format(timestamp, secret)
    string_to_sign_enc = string_to_sign.encode('utf-8')
    hmac_code = hmac.new(secret_enc, string_to_sign_enc, digestmod=hashlib.sha256).digest()
    sign = urllib.parse.quote_plus(base64.b64encode(hmac_code))
    # print(timestamp,sign)
    return timestamp,sign

# 钉钉机器人的token值
robot_token = {
    "ops": 'c3765d92d7b11d4c919f304a0a500ca653cf*********************'
}


# 定义一个钉钉告警函数
def alert_by_dingding(grpname, webhook_title, alert_message):
    if grpname in robot_token:
        try:
            token = robot_token[grpname]
            timestamp,sign = get_sign_timestamp()
            url = 'https://oapi.dingtalk.com/robot/send?access_token={0}&timestamp={1}&sign={2}'.format(token,timestamp,sign)


            webhook_header = {
                "Content-Type": "application/json",
                "charset": "utf-8"
            }
            webhook_message = {
                "msgtype": "markdown",
                "markdown": {
                    "title": webhook_title,
                    "text": alert_message
                }
            }

            sendData = json.dumps(webhook_message, indent=1)
            requests.post(url=url, headers=webhook_header, data=sendData)
        except Exception as e:
            traceback.print_exc(e,file=open('/tmp/alert_by_dingding.log', 'w+'))

def utc2local():
    # “”“UTC时间转本地时间（+8:00)
    offset = datetime.timedelta(hours=8)
    utc_time = datetime.datetime.utcnow()
    print(offset)
    local_st = utc_time + offset
    return local_st

def main():
    for app in app_dict:
        result = get_status(app_dict[app])
        if result:
            print("{0}应用已上架".format(app))
            continue
        else:
            print("{0}应用未上架,将发送报警信息".format(app))
            date_now = utc2local().strftime("%Y-%m-%d %H:%M:%S")
            description="未在Apple Store发现该应用，请检查"
            title = "检查应用上架情况"
            message = """
## 故障
**** 
**应用名称：** {appName}      
**应用包名：** {appKey}      
**告警描述：** {TRIGGER_DESCRIPTION}   
**告警时间：** {EVENT_DATE}   
**当前状态：** <font size="6" color="#ff3300">PROBLEM</font>
""".format(appName=app,appKey=app_dict[app],TRIGGER_DESCRIPTION=description,EVENT_DATE=date_now)
            alert_by_dingding('ops',title,message)

main()
```

