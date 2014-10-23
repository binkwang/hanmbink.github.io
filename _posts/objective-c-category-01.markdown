---
layout: post
title:  "OC特性之Category(一)"
date:   2014-10-23 16:00:00
categories: [技术,ios]
---

Category可以给原有的类增加新的方法，而不用重新建一个类，然后在原有的类的基础上使用这个方法。

另外注意：

* 不能增加数据成员

* 若Category添加的方法与原有的类的方法相同，那么原来的方法被覆盖


例子：给NSString类增加一个字符串反向输出的方法

`头文件`
{% highlight ruby %}
@interface NSString (ReverseString)
-(id)reverseString;//字符串反向输出
@end
{% endhighlight%}

`m文件`
{% highlight ruby %}
#import "NSString+ReverseString.h"
@implementation NSString (ReverseString)
-(id)reverseString
{
    NSUInteger len = [self length];
    NSMutableString *retStr = [NSMutableStringstringWithCapacity:len];
    while (len>0) {
        unichar c = [self characterAtIndex:--len];
        NSString *s = [NSString stringWithFormat:@"%C", c];
        [retStr appendString:s];
    }
    return retStr;
}
@end
{% endhighlight%}

`使用`
{% highlight ruby %}
#import "NSString+ReverseString.h"
-(void)test
{
    NSString *str = @"hello world!";
    NSString *retString = [str reverseString];
    NSLog(@"retString: %@", retString);
}
{% endhighlight%}

输出结果：`retString: !dlrow olleh`
