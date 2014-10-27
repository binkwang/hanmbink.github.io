---
layout: post
title:  "CGFloat和float的区别及案例分析"
date:   2014-10-27 17:00:00
categories: [技术,ios]
---

ASIHTTPRequest创建异步请求

{% highlight ruby %}
- (IBAction)grabURLInBackground:(id)sender
{
    NSURL *url = [NSURL URLWithString:@"http://allseeing-i.com"];
    ASIHTTPRequest *request = [ASIHTTPRequest requestWithURL:url];
    [request setDelegate:self];
    [request startAsynchronous];
}

- (void)requestFinished:(ASIHTTPRequest *)request
{
    // Use when fetching text data
    NSString *responseString = [request responseString];
    
    // Use when fetching binary data
    NSData *responseData = [request responseData];
}

- (void)requestFailed:(ASIHTTPRequest *)request
{
    NSError *error = [request error];
}
{% endhighlight %}


{% highlight ruby %}

{% endhighlight %}



{% highlight ruby %}

{% endhighlight %}




{% highlight ruby %}

{% endhighlight %}




{% highlight ruby %}

{% endhighlight %}
