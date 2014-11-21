---
layout: post
title:  "深入理解loadNibNamed"
date:   2014-11-21 16:00:00
categories: [技术,ios]
---

# 应用场景举例：加载自定义TableViewCell的xib文件

{% highlight ruby %}
NSArray *array = [[NSBundle mainBundle] loadNibNamed:@"CWTVCell4Stu" owner:self options:nil];
if ([[array lastObject] isKindOfClass:[CWTVCell4Stu class]]) {
    cell = (CWTVCell4Stu *)[array lastObject];
}
{% endhighlight %}


# owner参数

CWTVCell4Stu.xib文件的File’s Owner设置为哪个类，owner参数即为这个类的实例

例如：定义一个NSObject的子类FileOwner（作为xib的Owner），在FileOnwer中声明IBOutLet的关联变量：

{% highlight ruby %}
@interface FileOwner : NSObject
@property (nonatomic, weak) IBOutlet UIView *view;
@end
{% endhighlight %}

创建一个View.xib,xib的File’s Owner设为FileOwner类, 并建立view关联：View.xib中的根view指向FileOwner中的view变量

执行下面代码：

{% highlight ruby %}
FileOwner *owner = [FileOwner new];
[[NSBundle mainBundle] loadNibNamed:@"View" owner:owner options:nil];
{% endhighlight %}

则View.xib中视图树被加载到内存，并且owner.view指向根视图

# 需要理解的内容

* 获取的数组array是xib里的一级视图组成的数组，这个例子中，array里面只有一个对象：CWTVCell4Stu类的实例。所以[array lastObject]就是CWTVCell4Stu类的实例。

* loadNibNamed向CWTVCell4Stu.xib的视图树系统的每个view发送awakeFromNib消息，如果树状系统里面有自定义的view，则这个自定义view需要实现awakeFromNib方法做自身初始化。例如上面xib被加载的同时，创建xib的视图树保存到内存，各个视图的创建过程中都调用自己的awakeFromNib方法做自身初始化。根视图是自定义CWTVCell4Stu类型，所以CWTVCell4Stu.m要实现awakeFromNib方法。根视图下面的子视图都连接到CWTVCell4Stu类的outlet属性。

* 使用loadNibNamed来加载一个xib视图树，xib中的File’s Owner的类型为NSObject

# loadNibNamed与initWithNibName的异同

* 共同

都可以把xib文件加载到内存

* 区别

initWithNibName视图控制器的方法，用xib文件来初始化视图控制器的视图系统，需要把xib文件的File’s Owner设置为视图控制器类名，这样xib文件的视图树树就属于这个视图控制器。视图控制器的视图系统就用这个xib文件初始化。

{% highlight ruby %}
CWLoginViewController *loginViewController = [[CWLoginViewController alloc]initWithNibName:@"CWLoginViewController" bundle:nil];
{% endhighlight %}

loadNibNamed获取的是一个数组，里面元素是视图树；并且加载的xib文件的file’s owner类型为NSOjbect
