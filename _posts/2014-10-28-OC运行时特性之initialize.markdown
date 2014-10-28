---
layout: post
title:  "OC运行时特性之initialize"
date:   2014-10-28 10:30:00
categories: [技术,ios]
---

initialize不是init，在程序运行过程中，程序中每个类只调用一次initialize。

这个调用的时间发生在你的类接收到消息之前，但是在它的超类接收到initialize之后。

如果子类没有实现initialize，则会执行超类的initialize

举个例子，比如一个叫做Duck的类：

{% highlight ruby %}
#import "Duck.h";

@implementation Duck
+(void) initialize {
    NSLog(@"Duck initialize");
}

-(void) init {
    NSLog(@"Duck init");
}
@end
{% endhighlight %}

在这里记录initialize和init调用的时间, 建立三个Duck对象的实例：

{% highlight ruby %}
NSLog(@"Hello, World!");

Duck* duck1 = [[Duck alloc] init];

Duck* duck2 = [[Duck alloc] init];

Duck* duck3 = [[Duck alloc] init];
{% endhighlight %}

看一下记录：

[Session started at 2008-03-23 20:03:25 -0400.]

2008-03-23 20:03:25.869 initialize_example[30253:10b] Hello, World!

2008-03-23 20:03:25.871 initialize_example[30253:10b] Duck initialize

2008-03-23 20:03:25.872 initialize_example[30253:10b] Duck init

2008-03-23 20:03:25.873 initialize_example[30253:10b] Duck init

2008-03-23 20:03:25.873 initialize_example[30253:10b] Duck init

我们可以看到，虽然我们创建了3个Duck的实例，但是initialize仅仅被调用了一次。我们也可以看到，直到我们创建了一个Duck的实例，initialize才被调用。

但是如果Duck有一个子类的话，比如我们建一个Duck的子类叫做Chicken（好怪异……）

{% highlight ruby %}
#import <cocoa/Cocoa.h>

#import "Duck.h"

@interface Chicken : Duck {
    
}

@end
{% endhighlight %}

注意Chicken这个类并没有实现initialize方法。

如果我们同样运行这个程序，但是加上一个Chicken的实例：

{% highlight ruby %}
NSLog(@"Hello, World!");

Duck* duck1 = [[Duck alloc] init];
Duck* duck2 = [[Duck alloc] init];
Duck* duck3 = [[Duck alloc] init];

Chicken* chicken = [[Chicken alloc] init];
{% endhighlight %}

我们期待看到4个Duck的init调用（因为我们建立了3个Duck和一个Chicken），但是我们看到了这样情况：

[Session started at 2008-03-23 20:13:34 -0400.]

2008-03-23 20:13:34.696 initialize_example[30408:10b] Hello, World!

2008-03-23 20:13:34.698 initialize_example[30408:10b] Duck initialize

2008-03-23 20:13:34.699 initialize_example[30408:10b] Duck init

2008-03-23 20:13:34.700 initialize_example[30408:10b] Duck init

2008-03-23 20:13:34.700 initialize_example[30408:10b] Duck init

2008-03-23 20:13:34.700 initialize_example[30408:10b] Duck initialize

2008-03-23 20:13:34.701 initialize_example[30408:10b] Duck init

我们看到了4个Duck的init和2个Duck的initialize方法。这是怎么回事呢？

看来如果一个子类没有实现initialize方法，那么超类会调用这个方法两次，一次为自己，而一次为子类。

我们在Duck的initialize类中记录一下类名，这样可以看得更清楚：

{% highlight ruby %}
+(void) initialize {
    NSLog(@"Duck initialize class:%@",[self class]);
}
{% endhighlight %}

现在看明白了：

[Session started at 2008-03-23 20:21:08 -0400.]

2008-03-23 20:21:08.816 initialize_example[30513:10b] Hello, World!

2008-03-23 20:21:08.818 initialize_example[30513:10b] Duck initialize class:Duck

2008-03-23 20:21:08.819 initialize_example[30513:10b] Duck init

2008-03-23 20:21:08.820 initialize_example[30513:10b] Duck init

2008-03-23 20:21:08.820 initialize_example[30513:10b] Duck init

2008-03-23 20:21:08.820 initialize_example[30513:10b] Duck initialize class:Chicken

2008-03-23 20:21:08.821 initialize_example[30513:10b] Duck init

如果你希望确定只用了initialize一次用来实现某些单独运行的工作，或者希望实现仅仅运行一次的方法，检查一下[self class]，才能确定是否是你希望做到的效果。
