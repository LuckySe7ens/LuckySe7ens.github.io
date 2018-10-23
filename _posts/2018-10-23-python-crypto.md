---
layout: post
title: python3.5安装加密模块pycrypto
date: 2018-10-23
categories: blog
tags: [python,pycrypto]
description: 
---
python3.5安装加密模块pycrypto

需要研究AES加密，以为python自带AES模块，但让我失望的是目前的3.5版是不带的

参考手册应该使用A.M. Kuchling编写的pycrypto模块，官方网站http://www.pycrypto.org/

但下载后郁闷了，使用setup.py install命令后出错，需要vcvarsall.bat，缺c的编译器，模块的程序是用c实现的所以需要编译

再使用pip install pycrypto问题同样，查询教程需要安装visual C++或MinGW，现在已经抛弃那个巨大的软件了，安装过程也很复杂，

后来又想到直接下编译好的exe安装包，但很可惜，只有python3.3的，网址http://www.voidspace.org.uk



难道没有其他方法了吗

发现个好方法


执行
```
pip install --use-wheel --no-index --find-links=https://github.com/sfbahr/PyCrypto-Wheels/raw/master/pycrypto-2.6.1-cp35-none-win_amd64.whl pycrypto
```
执行编写的程序，没有发现模块
原来是大小写的问题
把Python安装目录\Lib\site-packages下的crypto改为Crypto就好了
