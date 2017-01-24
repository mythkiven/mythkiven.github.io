---
layout: post
title:  "NSURLProtocol小教程"
categories: Objective-C
tags: NSURLProtocol 缓存 
author: 3行代码
---

* content
{:toc}


iOS里面总有一些比较神秘，但也总是充满惊喜的API，譬如本博客持续更新的:NSURLProtocol。

NSURLProtocol并不是protocol而是一个虚拟类。使用的话需要实例化一个子类来操作，在这个子类中，可以实现如下的功能:

- 重定向网络请求
- 忽略网络请求，使用本地缓存
- 自定义网络请求的返回结果
- 全局的网络请求设置

注意:
- NSURLProtocol只能拦截 NSURLConnection、NSURLSession 和 UIWebView 中的请求，无法拦截WKWebView中发出的网络请求。
- 可以注册多个NSURLProtocol的子类，注册多个NSURLProtocol子类会逆序去执行，也就是先注册的子类后执行。
- AFN\SDWebImage等开始使用NSURLSession，需要注意其中潜在的坑
- UIWebView本身是有缓存的，基于NSURLCache来实现

如下是URL Loading System的组织图，本文只是简单的介绍下NSURLProtocol。


![](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/Art/nsobject_hierarchy_2x.png)

## 1、基本使用

#### 1、注册

``` java
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [NSURLProtocol registerClass:[CodeURLProtocol class]];
    return YES;
}
```
#### 2、实现方法 

// 1、每有一个请求的时候都会调用这个方法，如果返回YES就代表这个request需要被处理，反之就是不需要被处理。
``` java
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

// 2、这个方法返回request，可以修改request
``` java
+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request {
    return request;
}
```

// 3、这个方法用于处理被拦截的request，从而处理请求头、重定向、缓存等
``` js 
- (void)startLoading {

}
```

// 4、停止请求
``` js 
- (void) stopLoading {

}
```
#### 3、坑
不做解释，遇到的人都懂：
``` java
if ([NSURLProtocol propertyForKey:protocolKey inRequest:request]) {
    return NO;
}
[NSURLProtocol setProperty:@(YES) forKey:protocolKey inRequest:request];
```

#### 4、修改request请求头

``` java
- (void)startLoading {
    NSMutableURLRequest *request = [self.request mutableCopy];
    //给请求头添加一个请求体
    NSMutableDictionary *headers = [request.allHTTPHeaderFields mutableCopy];
    [headers setObject:@"3code.info" forKey:@"key"];
    request.allHTTPHeaderFields = headers;
    [NSURLProtocol setProperty:@(YES) forKey:protocolKey inRequest:request];
    //.....然后使用NSURLSession发送request
}
```
#### 5、request.URL重定向
将请求重定向到3code.info
```java

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
``` java
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
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask willCacheResponse:(NSCachedURLResponse *)proposedResponse completionHandler:(void (^)(NSCachedURLResponse *cachedResponse))completionHandler {
    completionHandler(proposedResponse);
}
```
NSURLConnection：
```java
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

上一期的项目有个需求，就是在移动网络下，不进行高清图片，视频等下载操作。其中遇到的问题就是拦截NSURLSession。这里简化为如下实现。

思路：使用NSURLProtocol拦截所有的request，过滤耗流量的request。并做处理。其实蛮简单的，但是AFNetwork、SDWebImageCache等三方库并未拦截。原因是NSURLSession。 
``` java

+ (BOOL)canInitWithRequest:(NSURLRequest *)request {
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

``` java
+ (NSURLSessionConfiguration *)zw_defaultSessionConfiguration {
//    如果需要NSURLProtocol来截获NSURLSession发出的请求，需要每一个NSURLSession在创建时配置的NSURLSessionConfiguration类的protocolClasses属性附上自定义的NSURLProtocol：如下
    NSURLSessionConfiguration *configuration = [self zw_defaultSessionConfiguration];
    NSArray *protocolClasses = @[[CodeURLProtocol class]];
    configuration.protocolClasses = protocolClasses;
    
    return configuration;
}
+ (void)load {
//  使用Method Swizzling方法，defaultSessionConfiguration实现（AFNetwork、SDWebImageCache在创建时使用的是[NSURLSessionConfiguration defaultSessionConfiguration]）如下：
    Method systemMethod = class_getClassMethod([NSURLSessionConfiguration class], @selector(defaultSessionConfiguration));
    Method zwMethod = class_getClassMethod([self class], @selector(zw_defaultSessionConfiguration));
    method_exchangeImplementations(systemMethod, zwMethod);
    [NSURLProtocol registerClass:[CodeURLProtocol class]];
}
```

以上就解决了NSURLProtocol拦截NSURLSession的问题。



## 3、缓存的实现 

实现这个的方式还是蛮多的，如果不想重复造轮子，就直接使用苹果的NSURLCache就足够了。

#### 缓存概述

- 1、NSURLCache，这是apple的网络请求缓存类，拿过来就能用。缓存在Library/Caches目录下。
- 2、自己缓存，在 ``` URLSession: task: didCompleteWithError: ``` 里面使用CoreData等缓存数据即可。

**下边就我使用过的CoreData、NSURLCache，分别作介绍：**

### 3.1、使用CoreData缓存：

1、创建 .xcdatamodeld 及实体类

有些没用过的可能不认识这个家伙，其实就是CoreData，将模型对象持久化到磁盘里。

- 创建实体： Add Entity
- 添加字段(键值)：data(Binary data)\encoding (String)\mimeType (String) \url(String)\timeStamp(data)
- 实体类：xcode菜单栏->Edit->Create NSManagedObject Subclass

注意： Codegen可能需要改成Manual/none。如果出现错误:```error: filename used twice```


2、NSURLProtocol中拦截缓存数据

```java
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveResponse:(NSURLResponse *)response completionHandler:(void (^)(NSURLSessionResponseDisposition))completionHandler {
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    completionHandler(NSURLSessionResponseAllow);
    self.response = response;
    self.mutableData = [[NSMutableData alloc] init];
}
-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
    [self.client URLProtocol:self didLoadData:data];
    [self.mutableData appendData:data];
}
-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
    if (error) {
        NSLog(@"error url = %@",task.currentRequest.URL.absoluteString);
        [self.client URLProtocol:self didFailWithError:error];
    } else {
        [self.client URLProtocolDidFinishLoading:self];
        // 缓存数据
        [self saveCachedResponse];
    }
}
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask willCacheResponse:(NSCachedURLResponse *)proposedResponse completionHandler:(void (^)(NSCachedURLResponse *cachedResponse))completionHandler {
    completionHandler(proposedResponse);
}
```

以及NSURLConnection：
```java
- (void) connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response {
    [self.client URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    self.response = response;
    self.mutableData = [[NSMutableData alloc] init];
}
- (void) connection:(NSURLConnection *)connection didReceiveData:(NSData *)data {
    [self.client URLProtocol:self didLoadData:data];
    [self.mutableData appendData:data];
}
- (void) connectionDidFinishLoading:(NSURLConnection *)connection {
    [self.client URLProtocolDidFinishLoading:self];
    // 缓存数据
    [self saveCachedResponse];
}
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error {
    [self.client URLProtocol:self didFailWithError:error];
}
```

保存方法如下，其中保存逻辑在AppDelegate中。
```java
#pragma mark 保存缓存到本地
- (void) saveCachedResponse {
    AppDelegate *delegate = [[UIApplication sharedApplication] delegate];
    NSManagedObjectContext *context = delegate.managedObjectContext;
    CachedURLResponse *cachedResponse = [NSEntityDescription insertNewObjectForEntityForName:@"CachedURLResponse"inManagedObjectContext:context];
    cachedResponse.data = self.mutableData;
    cachedResponse.url = self.request.URL.absoluteString;
    cachedResponse.timestamp = [NSDate date];
    cachedResponse.mimeType = self.response.MIMEType;
    cachedResponse.encoding = self.response.textEncodingName;
    
    NSError *error;
    [context save:&error];
    if (error) {
        NSLog(@"Could not cache the response.");
    } 
}
```

在 **startLoading**中调用如下方法：用于判断是否需要加载本地缓存
```java
-(void)localResource {
    CachedURLResponse *cachedResponse = [self cachedResponseForCurrentRequest];
    if (cachedResponse) {
        NSData *data = cachedResponse.data;
        NSString *mimeType = cachedResponse.mimeType;
        NSString *encoding = cachedResponse.encoding;
        NSURLResponse *response = [[NSURLResponse alloc] initWithURL:self.request.URL MIMEType:mimeType expectedContentLength:data.length
                                                    textEncodingName:encoding];
        
        [self.client URLProtocol:self didReceiveResponse:response
              cacheStoragePolicy:NSURLCacheStorageNotAllowed];
        
        [self.client URLProtocol:self didLoadData:data];
        [self.client URLProtocolDidFinishLoading:self];
    } else {
        NSMutableURLRequest *newRequest = [self.request mutableCopy];
        [NSURLProtocol setProperty:@YES forKey:protocolKey inRequest:newRequest];
        NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];
        NSOperationQueue *mainQueue = [NSOperationQueue mainQueue];
        NSURLSession *session = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:mainQueue];
        NSURLSessionDataTask *task = [session dataTaskWithRequest:newRequest];
        [task resume];
    }
}
```
检测本地是否有缓存：
```java
#pragma mark 检测本地是否存在缓存
- (NSCachedURLResponse *)cachedResponseForCurrentRequest {
    AppDelegate *delegate = [[UIApplication sharedApplication] delegate];
    NSManagedObjectContext *context = delegate.managedObjectContext;
    NSFetchRequest *fetchRequest = [[NSFetchRequest alloc] init];
    NSEntityDescription *entity = [NSEntityDescription entityForName:@"CachedURLResponse"inManagedObjectContext:context];
    [fetchRequest setEntity:entity];

    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"url == %@", self.request.URL.absoluteString];
    [fetchRequest setPredicate:predicate];

    NSError *error;
    NSArray *result = [context executeFetchRequest:fetchRequest error:&error];
    if (result && result.count > 0) {
        return result[0];
    }
    return nil;
}
```

AppDelegate：
```java
.h
// 缓存策略
@property (readonly, strong, nonatomic) NSManagedObjectContext *managedObjectContext;
@property (readonly, strong, nonatomic) NSManagedObjectModel *managedObjectModel;
@property (readonly, strong, nonatomic) NSPersistentStoreCoordinator *persistentStoreCoordinator;
- (void)saveContext;
- (NSURL *)applicationDocumentsDirectory;

.m

#import <CoreData/CoreData.h>

@synthesize managedObjectContext = _managedObjectContext;
@synthesize managedObjectModel = _managedObjectModel;
@synthesize persistentStoreCoordinator = _persistentStoreCoordinator;

- (void)applicationWillTerminate:(UIApplication *)application {
   [self saveContext];
}
- (void)saveContext{
    NSError *error = nil;
    NSManagedObjectContext *managedObjectContext = self.managedObjectContext;
    if (managedObjectContext != nil) {
        if ([managedObjectContext hasChanges] && ![managedObjectContext save:&error]) {
            NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
            abort();
        }
    }
}
- (NSManagedObjectContext *)managedObjectContext{
    if (_managedObjectContext != nil) {
        return _managedObjectContext;
    }
    NSPersistentStoreCoordinator *coordinator = [self persistentStoreCoordinator];
    if (coordinator != nil) {
        _managedObjectContext = [[NSManagedObjectContext alloc] init];
        [_managedObjectContext setPersistentStoreCoordinator:coordinator];
    }
    return _managedObjectContext;
}
- (NSManagedObjectModel *)managedObjectModel{
    if (_managedObjectModel != nil) {
        return _managedObjectModel;
    }
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"NSURLProtocolExample" withExtension:@"mom"];
    _managedObjectModel = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    return _managedObjectModel;
}
- (NSPersistentStoreCoordinator *)persistentStoreCoordinator{
    if (_persistentStoreCoordinator != nil) {
        return _persistentStoreCoordinator;
    }
    
    NSURL *storeURL = [[self applicationDocumentsDirectory] URLByAppendingPathComponent:@"NSURLProtocolExample.sqlite"];
    
    NSError *error = nil;
    _persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:[self managedObjectModel]];
    if (![_persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error]) {
        NSLog(@"Unresolved error %@, %@", error, [error userInfo]);
        abort();
    }
    return _persistentStoreCoordinator;
}
- (NSURL *)applicationDocumentsDirectory {
    return [[[NSFileManager defaultManager] URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject];
}
```

### 3.1、使用NSURLCache缓存：
与上边的CoreData原理一样，只不过使用了系统的。
下边给出主要的部分：

保存缓存
```java
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error{
    if (error) {
        [self.client URLProtocol:self didFailWithError:error];
    } else {
        [self.client URLProtocolDidFinishLoading:self];
        if (!self.data) {
            return;
        }
        // 保存缓存
        NSCachedURLResponse *cacheUrlResponse = [[NSCachedURLResponse alloc] initWithResponse:task.response data:self.data];
        [[NSURLCache sharedURLCache] storeCachedResponse:cacheUrlResponse forRequest:self.request];
        self.data = nil;
        NSLog(@"缓存成功");
    }
}

```

是否使用缓存
```java
- (void)startLoading{
    NSCachedURLResponse *urlResponse = [[NSURLCache sharedURLCache] cachedResponseForRequest:[self request]];
    if (urlResponse) {
        //如果缓存存在，则使用缓存。
        [self.client URLProtocol:self didReceiveResponse:urlResponse.response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
        [self.client URLProtocol:self didLoadData:urlResponse.data];
        [self.client URLProtocolDidFinishLoading:self];
        [self backgroundCheckUpdate];
        return;
    }
    NSMutableURLRequest *mutableRequest = [[self request] mutableCopy];
    [NSURLProtocol setProperty:@YES forKey:URLProtocolAlreadyHandleKey inRequest:mutableRequest];
    [self netRequestWithRequest:mutableRequest];
}
```

这种方式使用起来较为简便，没有太多的嵌入感。





