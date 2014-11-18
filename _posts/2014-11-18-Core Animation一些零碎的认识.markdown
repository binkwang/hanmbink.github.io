---
layout: post
title:  "Core Animation一些零碎的认识"
date:   2014-11-18 18:00:00
categories: [技术,ios]
---

# 动画

什么是Animation(动画),简单点说就是在一段时间内,显示的内容发生了变化.

对CALayer来说就是在一段时间内,其Animatable Property(可动画属性)发生了变化.

从CALayer(CA = Core Animation)类名来看就可以看出iOS的Layer就是为动画而生的,便于实现良好的交互体验.

这里涉及到两个东西: 一是Layer(基类CALayer),一是Animation(基于CAAnimation).

Animation作用于Layer.CALayer提供了接口用于给自己添加Animation.

用于显示的Layer本质上讲是一个Model,包含了Layer的各种属性值.

Animation则包含了动画的时间,变化,以及变化的速度.

# CALayer
UIView的职责在于界面的显示和界面事件的处理.

每一个View的背后都有一个layer(可以通过view.layer进行访问),layer是用于界面显示的.

CALayer属于QuartzCore框架,非常重要,但并没有想象中的那么好理解.

我们通常操作的用于显示的Layer在Core Animation这层的概念中其实担当的是数据模型Model的角色,它并不直接做渲染的工作.


# CALayer的渲染架构

Layer也和View一样存在着一个层级树状结构,称之为图层树(Layer Tree).

直接创建的或者通过UIView获得的(view.layer)用于显示的图层树,称之为模型树(Model Tree).

模型树的背后还存在两份图层树的拷贝,一个是呈现树(Presentation Tree),一个是渲染树(Render Tree).

呈现树可以通过普通layer(其实就是模型树)的layer.presentationLayer获得,而模型树则可以通过modelLayer属性获得(详情文档).

模型树的属性在其被修改的时候就变成了新的值,这个是可以用代码直接操控的部分;呈现树的属性值和动画运行过程中界面上看到的是一致的,而渲染树是私有的,无法访问到,渲染树是对呈现树的数据进行渲染.

为了不阻塞主线程,渲染的过程是在单独的进程或线程中进行的,所以Animation的动画并不会阻塞主线程.
