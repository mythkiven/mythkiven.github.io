---
layout: post
title:  "NSURLProtocol使用报告"
categories: NSURLProtocol
tags: NSURLProtocol 离线缓存 
author: 3行代码
---

* content
{:toc}

注意的是:本文写的比较早，一些情况没有考虑进来。像WKWebview，NSURLSession的支持等，下面会统一说明。

iOS里面总有一些比较神秘，但也总是充满惊喜的API，譬如今天分享的NSURLProtocol。

NSURLProtocol并不是protocol是一个虚拟类。使用的话需要实例化一个子类来操作。在这个子类中，可以实现如下的功能，是不是很神奇？

- 重定向网络请求
- 忽略网络请求，使用本地缓存
- 自定义网络请求的返回结果
- 全局的网络请求设置

注意:
- NSURLProtocol只能拦截 UIURLConnection、NSURLSession 和 UIWebView 中的请求，无法拦截WKWebView(网络基于WebKit内核)中发出的网络请求。(可使用WKNavigationDelegate处理)
- AFN，SDWebImage等开始使用NSURLSession，这个也需要注意
- 可以注册多个NSURLProtocol的子类，注册多个NSURLProtocol子类会逆序去执行，也就是先注册的子类后执行。


如下是URL Loading System的组织图，本文只是简单的介绍下NSURLProtocol，感兴趣的可以在文档中阅读相关的内容。

[https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165i](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165i)

图：

![](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Art/nsobject_hierarchy_2x.png)

## 1、基本使用

#### 1、注册


    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
        [NSURLProtocol registerClass:[CodeURLProtocol class]];
        return YES;
    }

#### 2、实现方法 

// 1、每有一个请求的时候都会调用这个方法，如果返回YES就代表这个request需要被处理，反之就是不需要被处理。
```
static NSString * const protocolKey = @"protocolKey";

+ (BOOL)canInitWithRequest:(NSURLRequest *)request {
    //防止无限循环，因为一个请求在被拦截处理过程中，也会发起一个请求，这样又会走到这里
    if ([NSURLProtocol propertyForKey:protocolKey inRequest:request]) {
        return NO;
    }
    // 对https请求做相关处理
    NSString *scheme = [[request URL] scheme];
    if ([scheme caseInsensitiveCompare:@"https"] == NSOrderedSame) {
        
        return YES;
    }
    return NO;
}
```

// 2、这个方法返回request，一般不做处理直接返回request
```
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request {
    return request;
}
```

///3 这个方法用于处理被拦截的request，拦截请求头、重定向、缓存处理等
```
/**
 * 开始请求
 */
- (void)startLoading {
    
}
```

// 4 停止请求

    - (void) stopLoading {}

#### 3、坑
不做解释，遇到的人都懂：
```
if ([NSURLProtocol propertyForKey:protocolKey inRequest:request]) {
    return NO;
}
[NSURLProtocol setProperty:@(YES) forKey:protocolKey inRequest:request];
```

#### 4、修改request请求头

```
- (void)startLoading {
    NSMutableURLRequest *request = [self.request mutableCopy];

    //给请求头添加一个请求体
    NSMutableDictionary *headers = [request.allHTTPHeaderFields mutableCopy];
    [headers setObject:@"3code.info" forKey:@"key"];
    request.allHTTPHeaderFields = headers;

    [NSURLProtocol setProperty:@(YES) forKey:protocolKey inRequest:request];

    .....然后使用NSURLSession发送request
}
```
## 5、request.URL重定向
将请求重定向到3code.info
```

- (void)startLoading {

    NSMutableURLRequest *request = [self.request mutableCopy];
    //request改为访问3code.info了
    request.URL = [NSURL URLWithString:@"http://www.3code.info"];
    
    [NSURLProtocol setProperty:@(YES) forKey:protocolKey inRequest:request];
    
    //使用NSURLSession继续把重定向的request发送出去
    NSURLSessionConfiguration *config = [NSURLSessionConfiguration ephemeralSessionConfiguration];
    NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
    
    NSURLSession *session = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:mainQueue];
    
    NSURLSessionDataTask *task = [session dataTaskWithRequest:request];
    
    [task resume];
}
```
#### 6、NSURLProtocolClient

webview中发送一个request，在这里拦截后使用NSURLSession重新发request。那webview是收不到response的。这里就要做一个处理，每一个NSURLProtocol的子类都有一个client对象来处理response。写法用下边这个就行，比较固定的。(保留了以前的NSURLConnection)

本次更新：
```
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
    if (error) {
        [self.client URLProtocol:self didFailWithError:error];
    } else {
        [self.client URLProtocolDidFinishLoading:self];
    }
}
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler {
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    
    completionHandler(NSURLSessionResponseAllow);
}
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
    [self.client URLProtocol:self didLoadData:data];
}
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask willCacheResponse:(NSCachedURLResponse *)proposedResponse completionHandler:(void (^)(NSCachedURLResponse *cachedResponse))completionHandler
{
    completionHandler(proposedResponse);
}
```
历史记录
```
- (void) connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
}
- (void) connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    [self.client URLProtocol:self didLoadData:data];
}
- (void) connectionDidFinishLoading:(NSURLConnection *)connection {
    [self.client URLProtocolDidFinishLoading:self];
}
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    [self.client URLProtocol:self didFailWithError:error];
}
```

## 2、NSURLProtocol拦截NSURLSession

上一期的项目有个需求，就是在移动网络下，不进行高清图片，视频等下载操作。其中遇到的问题就是拦截NSURLSession。这里简化以不下载图片为例，解说如何实现。

思路：使用NSURLProtocol拦截所有的request，过滤耗流量的request。并做处理。其实蛮简单的，但是AFNetwork、SDWebImageCache等三方库并未拦截。原因是NSURLSession。 
```

+ (BOOL)canInitWithRequest:(NSURLRequest *)request
{
    if ([NSURLProtocol propertyForKey:protocolKey inRequest:request]) {
        return NO;
    }
    //满足条件时直接截获请求
    if ([[UserConfig sharedInstance] isOffPicture] && ([request.URL.pathExtension caseInsensitiveCompare:@"jpg"] == NSOrderedSame || [request.URL.pathExtension caseInsensitiveCompare:@"png"] == NSOrderedSame) && [self isPictureDownload]) {
        return YES;
    }
    return NO;
}
```

从官方文档中可以了解到自定义的NSURLProtocol子类需要赋给NSURLSessionConfiguration的protocolClasses属性,同时其创建的Session时关联的NSURLSessionConfiguration为defaultSessionConfiguration。那么解决方法就跃然纸上了，如下代码：

```
+ (NSURLSessionConfiguration *)zw_defaultSessionConfiguration{
//    如果需要NSURLProtocol来截获NSURLSession发出的请求，需要每一个NSURLSession在创建时配置的NSURLSessionConfiguration类的protocolClasses属性附上自定义的NSURLProtocol：如下
    
    NSURLSessionConfiguration *configuration = [self zw_defaultSessionConfiguration];
    NSArray *protocolClasses = @[[CodeURLProtocol class]];
    configuration.protocolClasses = protocolClasses;
    
    return configuration;
}
+ (void)load{
//  使用Method Swizzling方法，defaultSessionConfiguration实现（AFNetwork、SDWebImageCache在创建时使用的是[NSURLSessionConfiguration defaultSessionConfiguration]）如下：
    Method systemMethod = class_getClassMethod([NSURLSessionConfiguration class], @selector(defaultSessionConfiguration));
    Method zwMethod = class_getClassMethod([self class], @selector(zw_defaultSessionConfiguration));
    method_exchangeImplementations(systemMethod, zwMethod);
    
    [NSURLProtocol registerClass:[CodeURLProtocol class]];
}

```

以上就解决了NSURLProtocol拦截NSURLSession的问题。



## 3、缓存策略

#### 1、创建.xcdatamodeld

有些没用过的可能不认识这个家伙，其实就是CoreData，将模型对象的持久化到磁盘里。

- 创建实体 Add Entity
- 添加字段(键值)data(Binary data)\encoding (String)\mimeType (String) \url(String)\timeStamp(data)

注意： Codegen可能需要改成Manual/none。如果出现错误:```error: filename used twice```

#### 2、创建操作实体类
xcode菜单栏->Edit->Create NSManagedObject Subclass





