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
#pragma mark init / dealloc

+ (void)initialize
{
	if (self == [ASIHTTPRequest class]) {
		persistentConnectionsPool = [[NSMutableArray alloc] init];
		connectionsLock = [[NSRecursiveLock alloc] init];
		progressLock = [[NSRecursiveLock alloc] init];
		bandwidthThrottlingLock = [[NSLock alloc] init];
		sessionCookiesLock = [[NSRecursiveLock alloc] init];
		sessionCredentialsLock = [[NSRecursiveLock alloc] init];
		delegateAuthenticationLock = [[NSRecursiveLock alloc] init];
		bandwidthUsageTracker = [[NSMutableArray alloc] initWithCapacity:5];
		ASIRequestTimedOutError = [[NSError alloc] initWithDomain:NetworkRequestErrorDomain code:ASIRequestTimedOutErrorType userInfo:[NSDictionary dictionaryWithObjectsAndKeys:@"The request timed out",NSLocalizedDescriptionKey,nil]];  
		ASIAuthenticationError = [[NSError alloc] initWithDomain:NetworkRequestErrorDomain code:ASIAuthenticationErrorType userInfo:[NSDictionary dictionaryWithObjectsAndKeys:@"Authentication needed",NSLocalizedDescriptionKey,nil]];
		ASIRequestCancelledError = [[NSError alloc] initWithDomain:NetworkRequestErrorDomain code:ASIRequestCancelledErrorType userInfo:[NSDictionary dictionaryWithObjectsAndKeys:@"The request was cancelled",NSLocalizedDescriptionKey,nil]];
		ASIUnableToCreateRequestError = [[NSError alloc] initWithDomain:NetworkRequestErrorDomain code:ASIUnableToCreateRequestErrorType userInfo:[NSDictionary dictionaryWithObjectsAndKeys:@"Unable to create request (bad url?)",NSLocalizedDescriptionKey,nil]];
		ASITooMuchRedirectionError = [[NSError alloc] initWithDomain:NetworkRequestErrorDomain code:ASITooMuchRedirectionErrorType userInfo:[NSDictionary dictionaryWithObjectsAndKeys:@"The request failed because it redirected too many times",NSLocalizedDescriptionKey,nil]];
		sharedQueue = [[NSOperationQueue alloc] init];
		[sharedQueue setMaxConcurrentOperationCount:4];

	}
}


- (id)initWithURL:(NSURL *)newURL
{
	self = [self init];
	[self setRequestMethod:@"GET"];

	[self setRunLoopMode:NSDefaultRunLoopMode];
	[self setShouldAttemptPersistentConnection:YES];
	[self setPersistentConnectionTimeoutSeconds:60.0];
	[self setShouldPresentCredentialsBeforeChallenge:YES];
	[self setShouldRedirect:YES];
	[self setShowAccurateProgress:YES];
	[self setShouldResetDownloadProgress:YES];
	[self setShouldResetUploadProgress:YES];
	[self setAllowCompressedResponse:YES];
	[self setShouldWaitToInflateCompressedResponses:YES];
	[self setDefaultResponseEncoding:NSISOLatin1StringEncoding];
	[self setShouldPresentProxyAuthenticationDialog:YES];
	
	[self setTimeOutSeconds:[ASIHTTPRequest defaultTimeOutSeconds]];
	[self setUseSessionPersistence:YES];
	[self setUseCookiePersistence:YES];
	[self setValidatesSecureCertificate:YES];
	[self setRequestCookies:[[[NSMutableArray alloc] init] autorelease]];
	[self setDidStartSelector:@selector(requestStarted:)];
	[self setDidReceiveResponseHeadersSelector:@selector(request:didReceiveResponseHeaders:)];
	[self setWillRedirectSelector:@selector(request:willRedirectToURL:)];
	[self setDidFinishSelector:@selector(requestFinished:)];
	[self setDidFailSelector:@selector(requestFailed:)];
	[self setDidReceiveDataSelector:@selector(request:didReceiveData:)];
	[self setURL:newURL];
	[self setCancelledLock:[[[NSRecursiveLock alloc] init] autorelease]];
	[self setDownloadCache:[[self class] defaultCache]];
	return self;
}

+ (id)requestWithURL:(NSURL *)newURL
{
	return [[[self alloc] initWithURL:newURL] autorelease];
}
{% endhighlight %}

`2 发起异步请求`

{% highlight ruby %}
- (void)startAsynchronous
{
#if DEBUG_REQUEST_STATUS || DEBUG_THROTTLING
ASI_DEBUG_LOG(@"[STATUS] Starting asynchronous request %@",self);
#endif
    [sharedQueue addOperation:self];
}

- (void)handleStreamComplete
{
    //省略很多
    [self requestFinished];
    //省略很多
}
// Subclasses might override this method to process the result in the same thread
// If you do this, don't forget to call [super requestFinished] to let the queue / delegate know we're done
- (void)requestFinished
{
#if DEBUG_REQUEST_STATUS || DEBUG_THROTTLING
	ASI_DEBUG_LOG(@"[STATUS] Request finished: %@",self);
#endif
	if ([self error] || [self mainRequest]) {
		return;
	}
	if ([self isPACFileRequest]) {
		[self reportFinished];
	} else {
		[self performSelectorOnMainThread:@selector(reportFinished) withObject:nil waitUntilDone:[NSThread isMainThread]];
	}
}

/* ALWAYS CALLED ON MAIN THREAD! */
- (void)reportFinished
{
	if (delegate && [delegate respondsToSelector:didFinishSelector]) {
		[delegate performSelector:didFinishSelector withObject:self];
	}

	#if NS_BLOCKS_AVAILABLE
	if(completionBlock){
		completionBlock();
	}
	#endif

	if (queue && [queue respondsToSelector:@selector(requestFinished:)]) {
		[queue performSelector:@selector(requestFinished:) withObject:self];
	}
}
{% endhighlight %}

未完待续..
