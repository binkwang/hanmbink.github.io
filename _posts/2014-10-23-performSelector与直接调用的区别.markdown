---
layout: post
title:  "performSelector与直接调用的区别"
date:   2014-10-23 22:40:00
categories: [技术,ios]
---

直接调用

{% highlight ruby %}
[delegate imageDownloader:self didFinishWithImage:image];
{% endhighlight %}

使用performSelector调用

{% highlight ruby %}
[delegate performSelector:@selector(imageDownloader:didFinishWithImage:) withObject:self withObject:image];
{% endhighlight %}

区别在于：

* performSelector是运行时系统负责去找方法的，在编译时候不做任何校验；如果直接调用编译时会自动校验。 

  如果`imageDownloader:didFinishWithImage:`方法不存在，那么直接调用在编译时候就能够发现，但是使用performSelector的话一定是在运行时候才能发现（此时程序崩溃）；

  Cocoa支持在运行时向某个类添加方法，即方法编译时不存在但是运行时候存在，这时候必然需要使用performSelector去调用。

  所以有时候如果使用了performSelector，为了程序的健壮性，会使用`respondsToSelector:`方法检查

  用下面的方式可以随时随地向任何对象发送任何消息，编译阶段不会报错，运行阶段也不会报错
  {% highlight ruby %}
  if ([delegate respondsToSelector:@selector(imageDownloader:didFinishWithImage:)]){    
      [delegate performSelector:@selector(imageDownloader:didFinishWithImage:) withObject:self withObject:image];
  }
  {% endhighlight %}

* 直接调用方法时候，一定要在头文件中声明该方法，也要将头文件import进来；而使用performSelector时候，可以不用import引入方法所在的头文件，直接用performSelector调用即可。

参考：

[http://blog.csdn.net/meegomeego/article/details/20041887][url01]

[url01]:  http://blog.csdn.net/meegomeego/article/details/20041887

