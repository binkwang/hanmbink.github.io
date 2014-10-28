---
layout: post
title:  "OC特性之Category(二)"
date:   2014-10-28 11:00:00
categories: [技术,ios]
---

Category不能添加成员变量,但能添加属性，本文介绍Category添加属性的方式

`方式一：`

给一个类声明一个属性，编译器会自动生成一个以下划线开头的实例变量；但是给类别添加属性，并不会为这个属性自动生成对应的成员变量

{% highlight ruby %}
@interface UIScrollView (MJRefresh)
@property (copy, nonatomic) NSString *headerRefreshingText;
@end

@implementation UIScrollView (MJRefresh)
- (void)setHeaderRefreshingText:(NSString *)headerRefreshingText
{
    self.header.refreshingText = headerRefreshingText;
}
- (NSString *)headerRefreshingText
{
    return self.header.refreshingText;
}
@end
{% endhighlight %}

这样添加的属性有如下特征：

1 自己实现set/get方法

2 不能在这个类别本身的实现中使用，只能在类别外使用，相当于给UIScrollView添加了属性

`方式二：使用Associative References`

{% highlight ruby %}
@interface UIScrollView()
@property (weak, nonatomic) MJRefreshHeaderView *header;
@end

@implementation UIScrollView (MJRefresh)
static char MJRefreshHeaderViewKey;
- (void)setHeader:(MJRefreshHeaderView *)header {
    [self willChangeValueForKey:@"MJRefreshHeaderViewKey"];
    objc_setAssociatedObject(self, &MJRefreshHeaderViewKey,header,OBJC_ASSOCIATION_ASSIGN);
    [self didChangeValueForKey:@"MJRefreshHeaderViewKey"];
}
- (MJRefreshHeaderView *)header {
    return objc_getAssociatedObject(self, &MJRefreshHeaderViewKey);
}
@end
{% endhighlight %}

这样添加的属性有如下特征：

1 属性可以在类别本身的实现中使用

2 在类别本身的实现中使用该属性时，只能通过self.header的方式访问，而不能通过_header访问 （.操作符其实就是set/get方法）

3 添加运行时：#import <objc/runtime.h>

4 添加的属性存储在对象的Associative References的dictionary中；添加的属性并不改变类对象的大小
