---
title: Install pyhs2 for windows
tags: [python,hive]
date: 2018-08-08
categories: 环境配置
---

在网站 [pythonlibs](https://www.lfd.uci.edu/~gohlke/pythonlibs/) 下载whl包
* sasl-0.2.1-cp27-cp27m-win_amd64.whl
* pyhs2-0.6.0-py2.py3-none-any.whl 

命令行安装
```shell
pip install sasl-0.2.1-cp27-cp27m-win_amd64.whl
pip install pyhs2-0.6.0-py2.py3-none-any.whl
```

在安装时可能发生pip版本低,需升级pip
```
pip install --upgrade pip
```

由于pip 升级后版本中没有main(),使用PyCharm IDE环境安装其他包会发生错误
> PyCharm安装目录\helpers\packaging_tool.py

在头部加上
```python
import pip._internal as pip_new
```

然后分别修改文件中的这两行中的pip
```python
return pip.main(['install'] + pkgs)
return pip.main(['uninstall', '-y'] + pkgs)
```
为
```python
return pip_new.main(['install'] + pkgs)
return pip_new.main(['uninstall', '-y'] + pkgs)
```
