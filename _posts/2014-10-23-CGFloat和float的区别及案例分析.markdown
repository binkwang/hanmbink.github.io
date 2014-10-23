---
layout: post
title:  "CGFloat和float的区别及案例分析"
date:   2014-10-23 23:00:00
categories: [技术,ios]
---

* 区别

  在32位下，CGFloat定义为float; 在64位下，CGFloat定义为double

  `typedef float CGFloat;// 32-bit`  

  `typedef double CGFloat;// 64-bit` 

* 编程策略
  
  对于需要兼容64位机器的程序而言，所有使用float的地方都改为用CGFloat。

  长远角度考虑推荐尽量使用CGFloat，尽管在32位上相比float增加了一些memory footprint的消耗。


* 案例分析

  64位系统下tableViewCell界面错位，原因如下：

  UITableView的cell高度代理函数:

  `- (CGFloat)tableView:(UITableView *)tableViewheightForRowAtIndexPath:(NSIndexPath *)indexPath`

  返回值类型是`CGFloat`，如果实现该方法时写成`float`，在32位不会警告而且界面显示正常。如果在64位下，则有警告：

  `Conflicting return type in implementation of 'tableView:heightForRowAtIndexPath:': 'CGFloat' (aka 'double') vs 'float' `

  而且运行起来界面上各个cell都错位了。

* 参考资料

  [http://blog.sina.com.cn/s/blog_6c6b2acd0100z4jb.html][url01]

[url01]:  http://blog.sina.com.cn/s/blog_6c6b2acd0100z4jb.html
