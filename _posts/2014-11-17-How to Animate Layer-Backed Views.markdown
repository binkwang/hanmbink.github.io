---
layout: post
title:  "How to Animate Layer-Backed Views"
date:   2014-11-17 18:00:00
categories: [技术,ios]
---

既然修改uiview的frame实际上在底层就是修改layer的frame,那么为什么修改view的frame属性不会以layer的隐式动画形式来改变，而在uiview的动画block中修改frame会以动画来改变？

两种类型的layer:

1 附加到 view 上的 layer, 即可以通过view.layer获取的layer。view有了layer的底层支持，所以view又称为“Layer-Backed Views”

2 单独的 layer

How to Animate Layer-Backed Views

当一个layer是view.layer时，这个layer可动画属性的改变不会用隐式动画的形式来展示，即layer的隐式动画不起作用。

原因如下：

UIView 默认情况下禁止了 layer 动画，但是在 animation block 中又重新启用了它们。

无论何时一个可动画的 layer 属性改变时，layer 都会寻找并运行合适的 'action' 来实行这个改变。action即CAAction，也就是动画。

当这个layer是某个view的支持时，layer按照这样的流程来寻找action:

layer 通过向它的 delegate 发送 actionForLayer:forKey: 消息来询问提供一个对应属性变化的 action。delegate可以通过返回以下三者之一来进行响应：

* 它可以返回一个动作对象，这种情况下 layer 将使用这个动作。

* 它可以返回一个 nil， 这样 layer 就会到其他地方继续寻找。

* 它可以返回一个 NSNull 对象，告诉 layer 这里不需要执行一个动作，搜索也会就此停止。

当 layer 在背后支持一个 view 的时候，view 就是它的 delegate；layer属性改变时 layer 会向 view 请求一个动作，而一般情况下 view 将返回一个 NSNull，只有当属性改变发生在动画 block 中时，view 才会返回实际的动作。

验证：

{% highlight ruby %}
NSLog(@"outside animation block: %@“, [myView actionForLayer:myView.layer forKey:@"position"]);
[UIView animateWithDuration:0.3 animations:^{
        NSLog(@"inside animation block: %@",[myView actionForLayer:myView.layer forKey:@"position"]);
}];
{% endhighlight %}

输出：
outside animation block: <null>

inside animation block: <CABasicAnimation: 0x8c2ff10]]>

可以看到在 block 外 view 返回的是 NSNull 对象，而在 block 中时返回的是一个 CABasicAnimation。

对于 附加到 view 上的 layer 来说，对动作的搜索只会到第一步为止。

对于单独的 layer 来说，剩余的四个步骤可以在 CALayer 的 actionForKey: 文档中找到。

