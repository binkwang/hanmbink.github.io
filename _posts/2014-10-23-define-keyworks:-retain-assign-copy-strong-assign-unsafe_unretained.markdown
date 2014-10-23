---
layout: post
title:  "retain/assign/copy/strong/assign/unsafe_unretained简单封装"
date:   2014-10-23 14:30:00
categories: [技术,ios]
---

{% highlight ruby %}
#ifndef MY_RETAIN
#if __has_feature(objc_arc)
    #define MY_RETAIN strong
#else
    #define MY_RETAIN retain
#endif
#endif
{% endhighlight %}

{% highlight ruby %}
#ifndef MY_ASSIGN
#if __has_feature(objc_arc_weak)
    #define MY_ASSIGN weak
#elif __has_feature(objc_arc)
    #define MY_ASSIGN unsafe_unretained
#else
    #define MY_ASSIGN assign
#endif
#endif
{% endhighlight %}

{% highlight ruby %}
#ifndef MY_COPY
#define MY_COPY copy
#endif
{% endhighlight %}

{% highlight ruby %}
//对象类型使用MY_RETAIN声明
@property (MY_RETAIN, nonatomic) NSURLConnection *connection;
@property (MY_RETAIN, nonatomic) NSMutableDictionary *fieldsToBePosted;
@property (MY_RETAIN, nonatomic) NSMutableArray *filesToBePosted;
{% endhighlight %}

{% highlight ruby %}
//NSString对象类型使用MY_COPY声明
@property (MY_COPY, nonatomic) NSString *username;
@property (MY_COPY, nonatomic) NSString *password;
{% endhighlight %}

{% highlight ruby %}
//基本数据类型使用MY_ASSIGN声明
@property (nonatomic, MY_ASSIGN) NSInteger startPosition;
@property (nonatomic, MY_ASSIGN) BOOL isCancelled;
{% endhighlight %}

