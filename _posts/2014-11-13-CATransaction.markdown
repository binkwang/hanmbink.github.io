---
layout: post
title:  "CATransaction"
date:   2014-11-13 18:00:00
categories: [技术,ios]
---

CATransaction 事务类

{% highlight ruby %}
@interface CATransaction : NSObject
{% endhighlight %}

CATransaction可以对多个layer的属性同时进行修改.它分隐式事务,和显式事务.

区分隐式动画和隐式事务：隐式动画通过隐式事务实现动画。

区分显式动画和显式事务：显式动画有多种实现方式，显式事务是一种实现显式动画的方式。

1 隐式事务

除显式事务外,任何对于CALayer属性的修改,都是隐式事务.这样的事务会在run-loop中被提交.

（直接修改CALayer属性，都会默认用动画的形式显示这个改变）

{% highlight ruby %}
- (void)viewDidLoad {
    layer=[CALayer layer];
    layer.bounds = CGRectMake(0, 0, 200, 200);
    layer.position = CGPointMake(160, 250);
    layer.backgroundColor = [UIColor redColor].CGColor;
    layer.borderColor = [UIColor blackColor].CGColor;
    layer.opacity = 1.0f;
    [self.view.layer addSublayer:layer];
    [super viewDidLoad];
}
-(IBAction)changeLayerProperty
{
    [CATransaction setDisableActions:NO];//设置变化动画过程是否显示，默认为YES不显示
    layer.cornerRadius = (layer.cornerRadius == 0.0f) ? 30.0f : 0.0f;//设置圆角
    layer.opacity = (layer.opacity == 1.0f) ? 0.5f : 1.0f;//设置透明度
}
{% endhighlight %}


2 显式事务

通过明确的调用begin,commit来提交动画

{% highlight ruby %}
[CATransaction begin];
[CATransaction setValue:(id)kCFBooleanFalse forKey:kCATransactionDisableActions];//显式事务默认开启动画效果,kCFBooleanTrue关闭
[CATransaction setValue:[NSNumber numberWithFloat:5.0f] forKey:kCATransactionAnimationDuration];//动画执行时间
anotherLayer.cornerRadius = (anotherLayer.cornerRadius == 0.0f) ? 30.0f : 0.0f;
layer.opacity = (layer.opacity == 1.0f) ? 0.5f : 1.0f;
[CATransaction commit];
{% endhighlight %}


3 事务嵌套

{% highlight ruby %}
[CATransaction begin];
[CATransaction begin];
[CATransaction setDisableActions:YES];
layer.cornerRadius = (layer.cornerRadius == 0.0f) ? 30.0f : 0.0f;
[CATransaction commit];//这个动画并不会立即执行，需要等最外层的commit
[NSThread sleepForTimeInterval:10];
[CATransaction setValue:(id)kCFBooleanFalse forKey:kCATransactionDisableActions];//显式事务默认开启动画效果,kCFBooleanTrue关闭
[CATransaction setValue:[NSNumber numberWithFloat:10.0f] forKey:kCATransactionAnimationDuration];//动画执行时间
anotherLayer.cornerRadius = (anotherLayer.cornerRadius == 0.0f) ? 30.0f : 0.0f;
[CATransaction commit];
{% endhighlight %}
