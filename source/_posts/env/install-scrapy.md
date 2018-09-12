---
title: Install Scrapy
tags: [scrapy]
date: 2016-06-30
categories: 环境配置
---

## 安装 python
```bash
$ sudo apt-get install python2.7
```
## 安装 setuptools
```bash
$ wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
```
## 安装 pip
```bash
$ sudo easy_install pip
```
## 安装 scrapy
```bash
$ sudo pip Scrapy
```
## 问题
- 安装scrapy出现, Python.h: No such file or directory

    解决方法是安装python-dev，这是Python的头文件和静态库包:
```bash
$ sudo apt-get install python-dev
```
