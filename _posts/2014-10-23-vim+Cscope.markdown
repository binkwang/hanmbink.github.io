---
layout: post
title:  "vim+cscope 代码编辑器"
date:   2014-10-23 14:00:00
categories: [技术,Linux]
---

简单介绍一下vim+cscope的使用

* vim必须先支持cscope，通过`#vim --version grep 'cscope'`命令来查看是否支持，如果不支持，需要重装vim

* 为代码生成一个cscope数据库。在项目根目录运行下面的命令： 

	`#cscope -Rbq`
 
这个命令会生成三个文件：`cscope.out, cscope.in.out, cscope.po.out`

其中cscope.out是基本的符号索引，后两个文件是使用”-q“选项生成的，可以加快cscope的索引速度。

Cscope在生成数据库中，在项目目录中未找到的头文件，会自动到/usr/include目录中查找，也可以是用参数组织从/usr/include目录中查找。

Cscope缺省只解析C文件(.c和.h)、lex文件(.l)和yacc文件(.y)，虽然它也可以支持C++以及Java，但它在扫描目录时会跳过C++及Java后缀的文件。

* 用vim打开一个.c文件,在vim中执行下面命令，来加入数据库文件：

`:cs add cscope.out`

如果是vim7.0版本，则不需要执行上面的命令，否则会提示`duplicate cscope database not added`错误

* 命令`:cs show`：查看加载的数据库文件

* 命令`:cs find s 函数或变量名`：查看这个名称出现的位置

* 命令`:cs find g 函数或变量名`：直接跳转到这个名称实现的位置
