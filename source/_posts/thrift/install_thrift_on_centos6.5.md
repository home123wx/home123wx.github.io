---
title: install Thrift on CentOS 6.5
tags: [thrift]
date: 2018-12-29
categories: 环境配置
---


##### 更新系统
```
yum update
```

##### 安装平台开发包
```
yum groupinstall "Development Tools"
```

##### 更新 autoconf
```
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz
tar xvf autoconf-2.69.tar.gz
cd autoconf-2.69
./configure --prefix=/usr
make
make install
cd ..
```

##### 更新automake
```
wget http://ftp.gnu.org/gnu/automake/automake-1.14.tar.gz
tar xvf automake-1.14.tar.gz
cd automake-1.14
./configure --prefix=/usr
make
make install
cd ..
```

##### 更新bison
```
wget http://ftp.gnu.org/gnu/bison/bison-2.5.1.tar.gz
tar xvf bison-2.5.1.tar.gz
cd bison-2.5.1
./configure --prefix=/usr
make
make install
cd ..
```

##### 添加C++依赖包
```
yum install libevent-devel zlib-devel openssl-devel
```

##### 安装boost
```
wget http://sourceforge.net/projects/boost/files/boost/1.53.0/boost_1_53_0.tar.gz
tar xvf boost_1_53_0.tar.gz
cd boost_1_53_0
./bootstrap.sh
./b2 install
```

##### 安装Thrift
```
git clone https://github.com/apache/thrift.git
cd thrift
./bootstrap.sh
./configure --with-lua=no
make
make install
```

##### 问题

```
# 错误提示
g++: error: /usr/local/lib/libboost_unit_test_framework.a: No such file or directory

# 解决
yum install boost-devel-static
# 执行make任出现该问题，后来发现安装位置不在 /usr/local/lib/

find / -name "libboost_unit_test_framework.a"
# 在 /usr/lib64/ 目录下
# 建立软连接
ln -s /usr/lib64/libboost_unit_test_framework.a /usr/local/lib/libboost_unit_test_framework.a

```