---
layout: post
title:  "ASIHTTPRequest源代码分析（一）"
date:   2014-10-27 17:00:00
categories: [技术,ios]
---

`ASIHTTPRequest创建异步请求`

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


`分析源代码`

`1 初始化请求`

{% highlight ruby %}
+ (void)initialize
{
	if (self == [ASIHTTPRequest class]) {
	//省略大量
	sharedQueue = [[NSOperationQueue alloc] init];
	[sharedQueue setMaxConcurrentOperationCount:4];

	}
}

- (id)initWithURL:(NSURL *)newURL
{
	self = [self init];
	//省略大量
	return self;
}

+ (id)requestWithURL:(NSURL *)newURL
{
	return [[[self alloc] initWithURL:newURL] autorelease];
}
{% endhighlight %}

`2 发起异步请求以及请求处理流程`

{% highlight ruby %}
- (void)startAsynchronous
{
#if DEBUG_REQUEST_STATUS || DEBUG_THROTTLING
ASI_DEBUG_LOG(@"[STATUS] Starting asynchronous request %@",self);
#endif
    [sharedQueue addOperation:self];
}

#pragma mark － 请求处理逻辑
// Create the request
- (void)main
{
	//省略大量
	[self startRequest];
	//省略大量
}

- (void)startRequest
{
   	// Start the HTTP connection
	CFStreamClientContext ctxt = {0, self, NULL, NULL, NULL};
    	if (CFReadStreamSetClient((CFReadStreamRef)[self readStream], kNetworkEvents, ReadStreamClientCallBack, &ctxt)) {
		if (CFReadStreamOpen((CFReadStreamRef)[self readStream])) {
			streamSuccessfullyOpened = YES;
		}
	}
}

#pragma mark － 回调，处理HTTP Response的主要函数 
- (void)handleNetworkEvent:(CFStreamEventType)type
{	
	//省略代码
    switch (type) {
        case kCFStreamEventHasBytesAvailable:
            [self handleBytesAvailable];
            break;
            
        case kCFStreamEventEndEncountered:
            [self handleStreamComplete];
            break;
            
        case kCFStreamEventErrorOccurred:
            [self handleStreamError];
            break;
            
        default:
            break;
    }
	//省略代码
}

- (void)handleBytesAvailable
{
	//省略代码
	[self readResponseHeaders];
	//省略代码
}

- (void)handleStreamComplete
{
	//省略代码
	[self readResponseHeaders];
	//省略代码
	[self requestFinished];
	//省略代码
}

#pragma mark － 解析HTTP response
- (void)readResponseHeaders
{
}

#pragma mark － 请求完成回调
- (void)requestFinished
{
	//省略代码
	[self performSelectorOnMainThread:@selector(reportFinished) withObject:nil waitUntilDone:[NSThread isMainThread]];
	//省略代码
}
- (void)reportFinished
{
	//省略代码
	[delegate performSelector:didFinishSelector withObject:self];
	//省略代码
}
{% endhighlight %}

未完待续..
