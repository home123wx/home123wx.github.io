---
title: CentOS 6.5 安装Scrapy
tags: [python,scrapy]
date: 2016-07-03
categories: 环境配置
---

### CentOS 6.5 Python升级到Python2.7
#### 下载源码 https://www.python.org/downloads/release/python-2712/
```bash
$ tar xvf Python-2.7.12.tgz
$ ./configure --prefix=/usr/local/python2.7.12
$ make && make install
```
#### 在/usr/bin/下建立符号链接指向安装好的python
```bash
ln -s /usr/local/python2.7.12/bin/python2.7 /usr/bin/python
ln -s /usr/local/python2.7.12/bin/python2.7-config /usr/bin/python-config
```
升级后yum不可用，修改/usr/bin/yum文件：
```bash
#!/usr/bin/python --> #!/usr/bin/python2.6
```
### 安装依赖包:
```bash
$ yum install python-lxml
$ yum install libxml2-devel
$ yum install libxslt-devel
```
### 安装setup-tools:
```bash
wget --no-check-certificate https://bootstrap.pypa.io/ez_setup.py -O - | python
ln -s /usr/local/python2.7.12/bin/easy_install-2.7 /usr/bin/easy_install
```
### 安装pip:
```bash
$ easy_install pip
$ pip install service-identity
$ pip install scrapy
```
### 测试scrapy:
```bash
scrapy bench
```
