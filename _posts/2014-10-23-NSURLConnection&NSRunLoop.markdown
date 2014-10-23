---
layout: post
title:  "NSURLConnection&NSRunLoop"
date:   2014-10-23 22:00:00
categories: [技术,ios]
---

主线程中创建一个NSURLConnection并异步执行

{% highlight ruby %}
[self performSelectorOnMainThread:@selector(start) withObject:nil waitUntilDone:YES];
- (void)start
{
    //step 1:请求地址
    NSURL *url = [NSURL URLWithString:@"www.2cto.com"];
    //step 2:实例化一个request
    NSURLRequest *request =[NSURLRequest requestWithURL:url cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:30.0];
    //step 3：创建链接
    self.connection = [[NSURLConnection alloc] initWithRequest:request delegate:self];//直接运行了
    if (self.connection) {
        NSLog(@"链接成功");
    }else {
        NSLog(@"链接失败");
    }
}
{% endhighlight %}

这样的异步请求存在的问题:

创建的NSURLConnection实例运行在主线程，主线程有一个运行的runloop实例来支持NSURLConnection的异步执行。

此时runloop的运行模式为NSDefaultRunLoopMode，这种mode下，如果主线程执行拖动操作，runloop不处理NSURLConnection的回调事件。

因为拖动操作发生过程中，使当前线程的runloop运行在UITrackingRunLoopMode模式下，这种模式下的runloop会暂定处理其他事件（异步请求回调、timer事件等）。


解决方法，修改代码:

{% highlight ruby %}
[self performSelectorOnMainThread:@selector(start) withObject:nil waitUntilDone:YES];
- (void)start
{
    //step 1:请求地址
    NSURL *url = [NSURL URLWithString:@"www.2cto.com"];
    //step 2:实例化一个request
    NSURLRequest *request =[NSURLRequest requestWithURL:url cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:30.0];
    //step 3：创建链接
    self.connection = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];//暂时不运行
    [connection scheduleInRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];//用NSRunLoopCommonModes
    [connection start];    
    if (self.connection) {
        NSLog(@"链接成功");
    }else {
        NSLog(@"链接失败");
    }
}
{% endhighlight %}

把NSURLConnection对象放在runloop的NSRunLoopCommonModes模式下执行，界面执行拖动时也会处理网络请求回调。
