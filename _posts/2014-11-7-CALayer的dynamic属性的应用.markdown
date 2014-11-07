---
layout: post
title:  "CALayer的dynamic属性的应用"
date:   2014-11-7 18:00:00
categories: [技术,ios]
---

CALayer 的 dynamic 属性提供了一种简单的机制来实现任何形式的动画。包括“不可视动画”。

什么是“不可视动画”，这个名词理解起来有点别扭，一般我们理解的动画是一个可视的过程，例如界面上正方形通过一个动画的形式在一定时间内从一个位置移动到另一个位置，这是我们容易理解的动画，也称为“可视动画”。

“不可视动画”就是不在界面上展示的变化，但是这个变化是用渐变的形式展现的，举个例子：声音的淡入淡出。（ps:图像的淡入淡出是个容易理解的可视动画）

这篇文章讲一下用CALayer 的 dynamic 属性实现声音的淡入淡出这个不可视动画。

# 1 子类化CALayer，设置声音属性volume
{% highlight ruby %}
@interface AudioLayer : CALayer
@property (nonatomic, assign) float volume;
@end
{% endhighlight %}

# 2 将属性volume标记为 @dynamic
{% highlight ruby %}
@dynamic volume;
{% endhighlight %}

# 3 覆写 +needsDisplayForKey: 方法, 用来通知CALayer有属性发生改变
{% highlight ruby %}

+(BOOL)needsDisplayForKey:(NSString *)key
{
    if ([key isEqualToString:@"volume"]) {
        NSLog(@"needsDisplayForKey");
        return YES;
    }
    return [super needsDisplayForKey:key];
}
{% endhighlight %}

# 4 覆写 -display 方法
{% highlight ruby %}
-(void)display
{
    NSLog(@"volume: %f", [[self presentationLayer] volume]);
    self.player.volume = [[self presentationLayer] volume];
}
{% endhighlight %}

# 5 为属性的变化做一个平滑的过渡动画
{% highlight ruby %}
-(id<CAAction>)actionForKey:(NSString *)key
{
    if ([key isEqualToString:@"volume"]) {
        NSLog(@"actionForKey");
        CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:key];
        animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionLinear];
        animation.fromValue = @([[self presentationLayer] volume]);
        animation.duration = 2.f;
        return animation;
    }
    return [super actionForKey:key];
}
{% endhighlight %}

以上就是dynamic属性的基本使用步骤，需要注意的是，当设置 CALayer 的某个属性时实际设置的是 model Layer 的值， 这里的 model Layer 表示正在进行的动画结束时 Layer 所达到的最终状态，也就是说如果在动画过程中取 model Layer 的值，获取到的是最终的值。presentation Layer 是 model Layer 的一个拷贝，但它的值所表示的是当前动画状态中属性的实时值。所以在actionForKey和display方法中，要使用presentation Layer中的属性值。

* 整个工程代码在这里下载: [https://github.com/hanmbink/DynamicProperty](https://github.com/hanmbink/DynamicProperty)








