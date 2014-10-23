---
layout: post
title:  "iOS的观察者模式之KVC&KVO"
date:   2014-10-23 18:30:00
categories: [技术,ios]
---

ios实现的观察者模式有两种：NSNotification和KVO。这篇文章介绍的是KVO。

KVC

NSKeyValueCoding，提供一种机制来间接访问对象的属性。是KVO的基础。

一个对象拥有某些属性。比如说，一个 Person 对象有一个 name 和一个 address 属性。

以 KVC 说法，Person 对象分别有一个 value 对应他的 name 和 address 的 key。 

key 只是一个字符串（即属性名），它对应的value可以是任意类型的对象。

从最基础的层次上看，KVC 有两个方法：一个是设置 key 的值，另一个是获取 key 的值。

{% highlight ruby %}
NSString *originalName = [person valueForKey:@"name"];// using the KVC accessor (getter) method 
[person setValue:newName forKey:@"name"];// using the KVC  accessor (setter) method. 
{% endhighlight %}

现在，如果 Person有另外一个 key配偶（spouse），spouse的 key值是另一个 Person对象，用 KVC可以这样写： 

{% highlight ruby %}
NSString *personsName = [p valueForKey:@"name"];// just using the accessor again, same as example above 
NSString *spousesName = [p valueForKeyPath:@"spouse.name"];// a "key path" instead of a normal "key" 
{% endhighlight %}

key 与 key path要区分开来，key可以从一个对象中获取值，而 key path可以将多个 key用点号 “.”分割连接起来，比如：

{% highlight ruby %}
[p valueForKeyPath:@"spouse.name"];
{% endhighlight %}

相当于这样： 

{% highlight ruby %}
[[p valueForKey:@"spouse"] valueForKey:@"name"];
{% endhighlight %}

以上是 KVC的基本知识

KVO

作用

通过 key path观察对象的值，当值发生变化的时候会收到通知

在iOS中，KVO最主要的目的还是实现两个对象之间的交互，比如都需要根据属性的变化来更新UI，

不用KVO，需要在每个更新属性的时候加上UI更新的代码。

用KVO，只需要添加一处UI更新的代码，因为KVO代码会自动的跟踪属性的变化，当变化的时候，会自己调用同一个变化的方法来处理，减少代码的冗余。


使用流程：
* 注册，指定被观察者的属性
* 实现回调方法
* 移除观察

实例 

假设一个场景,股票的价格显示在当前屏幕上，当股票价格更改的时候，实时显示更新其价格。 

定义DataModel

{% highlight ruby %}
@interface HRKVOModel : NSObject
@property(nonatomic,copy)NSString *stockName;
@property(nonatomic,copy)NSString *price;
@end
{% endhighlight %}

定义此model为viewController的属性，实例化它，监听它的属性，并在view上显示属性值

{% highlight ruby %}
@property(nonatomic, strong)StockData *stock;
- (void)viewDidLoad
{
    [super viewDidLoad];
    
    _stock = [[StockData alloc] init];
    [_stock setValue:@"searph" forKey:@"stockName"];
    [_stock setValue:@"10.0" forKey:@"price"];
    [_stock addObserver:self forKeyPath:@"price" options:NSKeyValueObservingOptionNew|NSKeyValueObservingOptionOld context:NULL];//NSObject的注册观察者方法
    
    myLabel = [[UILabel alloc]initWithFrame:CGRectMake(100, 100, 100, 30 )];
    myLabel.textColor = [UIColor redColor];
    myLabel.text = [stockForKVO valueForKey:@"price"];
    [self.view addSubview:myLabel];
    
    UIButton * b = [UIButton buttonWithType:UIButtonTypeRoundedRect];
    b.frame = CGRectMake(0, 0, 100, 30);
    [b addTarget:self action:@selector(buttonAction) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:b];
}
{% endhighlight %}

当点击button的时候，调用buttonAction方法，修改对象的属性

{% highlight ruby %}
-(void) buttonAction
{
    [_stock setValue:@"20.0" forKey:@"price"];
}
{% endhighlight %}

实现回调方法

{% highlight ruby %}
// whenever an observed key path changes, this method will be called 
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    if([keyPath isEqualToString:@"price"])
    {
        myLabel.text = [_stock valueForKey:@"price"];
    }
}
{% endhighlight %}

增加观察与取消观察是成对出现的，所以需要在最后的时候，移除观察者 

{% highlight ruby %}
- (void)dealloc
{
    [super dealloc];
    [_stock removeObserver:self forKeyPath:@"price"];
    [_stock release];
}
{% endhighlight %}

KVO与Notification的区别：

实现两个对象之间的交互用Notification也可以，区别在于：

Notification不是严格意义上的两个对象的交互，中间有一个NotificationCenter来作为中间人来进行沟通，KVO就纯粹是连个对象之间的交互了。

两者的相同点是都需要在最后释放注册的Object。


参考：

[1][url01]

[2][url02]

[3][url03]

[url01]:    http://magicalboy.com/kvc_and_kvo/
[url02]:    http://blog.csdn.net/messageloop3/article/details/8634798
[url03]:    http://deeryrl.blog.163.com/blog/static/15254287420123235434553/
