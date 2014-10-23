---
layout: post
title:  "NSObject的isEqual:方法"
date:   2014-10-23 23:10:00
categories: [技术,ios]
---

isEqual:比较两个对象的地址是否相同、类型是否相同、类里面的值是否相同，全部相同才返回真

{% highlight ruby %}
NSString *str1 = [[NSString alloc]initWithFormat:@"%@", @"11"];
NSString *str2 = [[NSString alloc]initWithFormat:@"%@", @"11"];
if ([str1 isEqual:str2]) {
    NSLog(@"NSString equal");
}else{
    NSLog(@"NSString not equal");
}
NSDictionary *dic1 = [[NSDictionary alloc]initWithObjectsAndKeys:@"111",@"key1",@"222",@"key2",nil];
NSDictionary *dic2 = [[NSDictionary alloc]initWithObjectsAndKeys:@"111",@"key1",@"222",@"key2",nil];
if ([dic1 isEqual:dic2]) {
    NSLog(@"NSDictionary equal");
}else{
    NSLog(@"NSDictionary not equal");
}
{% endhighlight %}

输出结果：

2014-10-17 17:40:01.756 DaWenXun[13877:60b] NSString equal

2014-10-17 17:40:01.759 DaWenXun[13877:60b] NSDictionary equal

`结论：NSString和NSDictionary肯定重写了isEqual方法`

自定义Person类：

{% highlight ruby %}
@interface Person : NSObject
@property(nonatomic,strong)NSString *name;
@property(nonatomic,strong)NSString *age;
-(id)initWithName:(NSString*)name andAge:(NSString*)age;
@end

@implementation Person
-(id)initWithName:(NSString*)name andAge:(NSString*)age
{
    self = [super init];
    if (self) {
        _name = name;
        _age = age;
    }
    return self;
}
@end
{% endhighlight %}

创建两个对象并比较：

{% highlight ruby %}
Person *person1 = [[Person alloc]initWithName:@"wang" andAge:@"30"];
Person *person2 = [[Person alloc]initWithName:@"wang" andAge:@"30"];
if ([person1 isEqual:person2]) {
    NSLog(@"Person equal");
}else{
    NSLog(@"Person not equal");

}
NSDictionary *dic1 = [[NSDictionary alloc]initWithObjectsAndKeys:person1,@"key1",@"222",@"key2",nil];
NSDictionary *dic2 = [[NSDictionary alloc]initWithObjectsAndKeys:person2,@"key1",@"222",@"key2",nil];
if ([dic1 isEqual:dic2]) {
    NSLog(@"NSDictionary equal");
}else{
    NSLog(@"NSDictionary not equal");
}
{% endhighlight %}

输出结果：

2014-10-17 17:43:02.639 DaWenXun[13895:60b] Person not equal

2014-10-17 17:43:02.643 DaWenXun[13895:60b] NSDictionary not equal

`结论：因为两个Person对象的地址不同，即使两个对象类型和属性值都线通，isEqual方法返回仍为NO`

现在在Person中重写父类的isEqual方法：

{% highlight ruby %}
@interface Person : NSObject
@property(nonatomic,strong)NSString *name;
@property(nonatomic,strong)NSString *age;
-(id)initWithName:(NSString*)name andAge:(NSString*)age;
@end

@implementation Person
-(id)initWithName:(NSString*)name andAge:(NSString*)age
{
    self = [super init];
    if (self) {
        _name = name;
        _age = age;
    }
    return self;
}
//重写isEqual方法
- (BOOL)isEqual:(id)other {
    if (other == self)
        return YES;
    if (!other || ![other isKindOfClass:[self class]])
        return NO;
    return [self isEqualToWidget:other];
}
- (BOOL)isEqualToWidget:(Person *)aWidget {
    if (self == aWidget)
        return YES;
    if (![(id)[self name] isEqual:[aWidget name]])
        return NO;
    if (![[self age] isEqual:[aWidget age]])
        return NO;
    return YES;
}
@end
{% endhighlight %}

创建对象并比较：

{% highlight ruby %}
Person *person1 = [[Person alloc]initWithName:@"wang" andAge:@"30"];
Person *person2 = [[Person alloc]initWithName:@"wang" andAge:@"30"];
if ([person1 isEqual:person2]) {
    NSLog(@"Person equal");
}else{
    NSLog(@"Person not equal");
}
NSDictionary *dic1 = [[NSDictionary alloc]initWithObjectsAndKeys:person1,@"key1",@"222",@"key2",nil];
NSDictionary *dic2 = [[NSDictionary alloc]initWithObjectsAndKeys:person2,@"key1",@"222",@"key2",nil];
if ([dic1 isEqual:dic2]) {
    NSLog(@"NSDictionary equal");
}else{
    NSLog(@"NSDictionary not equal");
}
{% endhighlight %}

输出结果：

2014-10-17 18:11:47.522 DaWenXun[13992:60b] Person equal

2014-10-17 18:11:47.524 DaWenXun[13992:60b] NSDictionary equal


参考资料

[http://blog.csdn.net/frankwun/article/details/8582747][url01]

[url01]:  http://blog.csdn.net/frankwun/article/details/8582747
