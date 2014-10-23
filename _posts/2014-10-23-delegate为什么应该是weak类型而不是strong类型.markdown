---
layout: post
title:  "delegate为什么应该是weak类型而不是strong类型"
date:   2014-10-23 22:50:00
categories: [技术,ios]
---

* 原因1 

  delegate没有必要是strong 是A负责创建B的，A的生命周期一定比B要长（B存在，A一定也存在；A存在，B不一定存在） 也就是说 在B（实例b）存在的时候，A（的实例）一定存在， 所以没有必要用retain将引用计数+1，assign足以 

* 原因2 

  退一步假设存在这样的情况：A比B先挂掉（A的实例先被release销毁） 那么当然希望是[a relase]后，A中的方法不再被B调用，但是使用retain时候，A还是没有被销毁，so A中的方法仍会被B调用，但是assgin就无此问题 

* 原因3 

  避免循环引用 两个类互相强引用，referance counting永远不会是0，一直会占用内存，如果delegate是弱引用，那delegate释放掉之后，这个类也可以被释放

`循环引用`

对象a创建并引用了对象b.对象b创建并引用了对象c.对象c创建并引用了对象b.

这时候b和c的引用计数分别是2和1。当a不再使用b，调用release释放对b的所有权，因为c还引用了b，所以b的引用计数为1，b不会被释放。b不释放，c的引用计数就是1，c也不会被释放。从此，b和c永远留在内存中。

`打断循环引用`

如果c引用b的时候是用weak类型弱引用，那么a释放b时，b的引用计数器变为0，b被释放，c也被释放。

一个例子：

UITableViewController对象a通过retain获取了UITableView对象b的所有权，这个UITableView对象b的delegate又是a，如果这个delegate是retain的，那基本上就没有机会释放这两个对象了。 

循环引用而产生的内存泄露是Instrument无法发现的。

参考：

iOS 5 ARC完全指南.pdf 

[http://blog.csdn.net/diyagoanyhacker/article/details/6591593][url01]

[http://www.cocoachina.com/ask/questions/show/93387][url02]

[url01]:  http://blog.csdn.net/diyagoanyhacker/article/details/6591593
[url02]:  http://www.cocoachina.com/ask/questions/show/93387
