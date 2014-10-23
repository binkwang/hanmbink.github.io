---
layout: post
title:  "Objective C 2.0的属性访问器和相应的编程策略"
date:   2014-10-23 15:00:00
categories: [技术,ios]
---

# object-c 2.0新特性－属性访问器

摘自《Objective-C 2.0的新特性与运行时编程》

属性访问器：不需要手动书写getter/setter方法，增加了property/synthesize关键字来实现此功能

`@property`

用@prpperty关键字声明属性，这样在编译后的代码中，自动添加成员变量的getter/setter方法

使用不同的参数，生成的setter方法是不一样的，例如：

* 属性是对象类型，需要使用retian/assign/copy参数，表示setter方法内部，持有对象的方式，retian增加引用计数器，是强引用；assign是变量直接赋值，是弱引用；copy是把setter的参数复制一份，再赋给成员变量

* 属性为基本数据类型时，不需要使用retian/assign/copy参数

* 所有类型的属性都可以使用atomic/nomatomic参数，前者表示属性是原子的，支持多线程并发访问，在setter方法中增加了同步锁，后者是非原子的，适合在非多线程的环境中提升效率，因为在setter中没有同步锁的代码

* 使用readonly参数不生成setter方法

`@synthesize`

表示在m文件中自动实现访问器方法

`点操作符`

使用点操作符号来引用对象的属性时，调用的是setter/getter方法，这也是object-c 2.0的新特性

“对象.属性”在“=”左侧，调用setter方法，例如self.name=@"Tom";  相当于[self setName:@"Tom"];

"对象.属性"在“=”右侧，调用getter方法

"对象.属性”作为一个参数时，调用getter方法

# 相应的编程策略

{% highlight ruby %}
@property (nonatomic, MY_RETAIN) NSArray *arr;  //1
- (void)initDatas
{
    NSMutableArray *array = [[NSMutableArray alloc]initWithCapacity:10];   
    for (NSUInteger i=0; i<3; i++) {
        NSNumber *num = [[NSNumber alloc]initWithInteger:i];
        [array addObject:num];
        [num release];
    }
    self.arr = array;             //2
    RELEASE_SAFELY(array);
    NSLog(@"_arr: %@", _arr);     //3
}
- (void)dealloc
{
    RELEASE_SAFELY(_arr);          //4
    [super dealloc];
}
{% endhighlight%}

* 声明属性时，对象属性用retain参数，这样在生成的setter方法里面会自动为属性增加引用计数器

* 给属性赋值，要另外创建一个相同类型的对象，然后用self.arr=array;方式来赋值，然后释放创建的对象。

如果用下面代码直接给属性创建空间：

`self.arr = [[NSMutableArray alloc]initWithCapacity:10];`

则_arr指向的内存区引用计数器为2，用alloc增加1次，用setter方法又增加1次。这样显得混乱，容易导致内存泄漏。

当然可以用下面赋值语句：

`_arr = [[NSMutableArrayalloc]initWithCapacity:10];`

这样，_arr指向的内存区引用计数器为1，只用alloc增加1次，没有调用setter方法。

* 使用属性时，用"_arr"形式，这样时直接访问对象本身，也可以用"self.arr"的形势，这样时调用getter方法，速度慢

* 在dealloc方法中释放属性

总之，给属性赋值的时候，统一用"self.属性名＝"的方式，然后在dealloc方法中把属性所指内存区的引用计数器减1

给属性赋值时，创建的临时对象，如果是用new,alloc,copy创建的，则赋值之后就release; 如果是用便利构造器创建的，则不需要release

调用属性时，采用"_属性名"的形式，而不是用“self.属性名”来间接调用。 

