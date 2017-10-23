---
layout: post
title:  "NSURLCache缓存的那些事"
categories: Objective-C
tags: NSURLCache 缓存
author: 3行代码
---

* content
{:toc}

不少的开源项目的网络缓存都是自己家的轮子(譬如AFN俩独立缓存模块：AFAFImagecache (处理image)、NSURLCache(默认URL缓存))，我们现在的项目并未涉及到太复杂的缓存需求，自己造轮子不划算直接用系统的NSURLCache就OK。

在使用中也遇到不少坑：

- 响应内容过大，不会被缓存；
- Transfer-Encoding: Chunked  分块传输也会导致不缓存。
- ...

目前在协助子公司开发的一个项目里涉及到各种HTML5中的离线存储需求，实际用的是NSURLProtocol+NSURLCache，根据场景，用两套缓存的方案。对NSURLProtocol感兴趣的可以浏览我的另一篇关于NSURLProtocol的blog。

### 基础篇

#### 1、扫盲普及

1、在iOS中，可以使用NSURLCache类缓存数据，iOS 5之前只支持内存缓存；从iOS 5开始同时支持内存缓存和硬盘缓存；iOS6.0开始，支持HTTPS缓存。

2、缓存分为:内存缓存\硬盘缓存。从app运行角度来说：如果程序没有退出，那么会使用内存中的缓存，如果已经退出重启，则使用硬盘缓存(读取到内存中)。

3、NSURLCache不支持Post请求的缓存处理，GET默认缓存：由于GET请求一般用来查询数据，POST请求一般是发大量数据给服务器处理。因此一般只对GET请求进行缓存，而不对POST请求进行缓存。

4、使用前提：

- 经常更新：不能用缓存；
- 一成不变：果断用缓存；
- 偶尔更新：可以定期更改缓存策略或者清除缓存
- GET网络请求、webview页面：果断缓存；
- App离线缓存功能等

5、使用也很简单，几行代码搞定：

```
// 1 修改默认的缓存空间：
- (BOOL)application:(UIApplication *)application
didFinishLaunchingWithOptions:(NSDictionary *)launchOptions{
  NSURLCache *URLCache = [[NSURLCache alloc] initWithMemoryCapacity:4 * 1024 * 1024diskCapacity:20 * 1024 * 1024 diskPath:nil];
  [NSURLCache setSharedURLCache:URLCache];
}
// 2 设置缓存策略：
request.cachePolicy = NSURLRequestReturnCacheDataElseLoad;
OK，搞定。
```
如果不想使用缓存，上边数值设置0即可。

6、缓存的有效性，下文会提到，服务器返回的数据需要一个有效期，过期有效期就得更新本地的缓存。


#### 2、基础：GET 缓存

1、基本用法:
```

（1）获得全局缓存对象 NSURLCache *cache = [NSURLCache sharedURLCache]; 
（2）设置内存缓存的最大容量（字节为单位，默认为512KB）- (void)setMemoryCapacity:
（3）设置硬盘缓存的最大容量（字节为单位，默认为10M）- (void)setDiskCapacity:
（4）硬盘缓存的位置：沙盒/Library/Caches
（5）取得某个请求的缓存- (NSCachedURLResponse *)cachedResponseForRequest:
（6）清除某个请求的缓存- (void)removeCachedResponseForRequest:
（7）清除所有的缓存- (void)removeAllCachedResponses;
```

2、缓存
```

    // 1.创建请求
    NSURL *url = [NSURL URLWithString:@"GET"];
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    request.cachePolicy = NSURLRequestReturnCacheDataElseLoad;
    
    // 2.根据request检测缓存  
    NSURLCache *cache = [NSURLCache sharedURLCache];
    NSCachedURLResponse *response = [cache cachedResponseForRequest:request];
    if (response) {
        NSLog(@"已经有缓存:%@",response.data);
        // 判断时间，超时就移除缓存
    } else {
        NSLog(@"暂无缓存,开启请求");
        [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError) {
         if (data) {
            NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableLeaves error:nil];
             NSLog(@"%@", dict);
         }
    }];
    }
    
    
```

#### 3、进阶：POST 缓存

POST 实现缓存需要自己手动实现，如下解说：

```

// 读取缓存
+ (id)getCacheResponseWithURL:(NSString *)url params:(NSDictionary *)params {
    id cacheData = nil;
    if (url) {
        // 指定目录
        NSString *directoryPath = DIRECTORYPATH;
        // url+param确定唯一路径
        NSString *originString = [NSString stringWithFormat:@"%@+%@",url,params];

        NSString *path = [directoryPath stringByAppendingPathComponent:[self md5:originString]];
        NSData *data = [[NSFileManager defaultManager] contentsAtPath:path];
        if (data) {
            cacheData = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableContainers error:nil];
        }
    }
    return cacheData;
}
// 写入缓存
+ (void)cacheResponseObject:(id)responseObject request:(NSURLRequest *)request params:(NSDictionary *)params {
    if (request && responseObject && ![responseObject isKindOfClass:[NSNull class]]) {
        NSString *directoryPath = DIRECTORYPATH;
        NSError *error = nil;
        if (![[NSFileManager defaultManager] fileExistsAtPath:directoryPath isDirectory:nil]) {
            [[NSFileManager defaultManager] createDirectoryAtPath:directoryPath
                                      withIntermediateDirectories:YES
                                                       attributes:nil
                                                            error:&error];
        }

        NSString *originString = [NSString stringWithFormat:@"%@+%@",request.URL.absoluteString,params];

        NSString *path = [directoryPath stringByAppendingPathComponent:[self md5:originString]];
        NSDictionary *dict = (NSDictionary *)responseObject;

        NSData *data = nil;
        if ([dict isKindOfClass:[NSData class]]) {
            data = responseObject;
        } else {
            data = [NSJSONSerialization dataWithJSONObject:dict
                                                   options:NSJSONWritingPrettyPrinted
                                                     error:&error];
        }
        if (data && error == nil) {
            [[NSFileManager defaultManager] createFileAtPath:path contents:data attributes:nil];
        }
    }
}
// 加密
+ (NSString *)md5:(NSString *)string {
    if (string == nil || [string length] == 0) {
        return nil;
    }
    unsigned char digest[CC_MD5_DIGEST_LENGTH], i;
    CC_MD5([string UTF8String], (int)[string lengthOfBytesUsingEncoding:NSUTF8StringEncoding], digest);
    NSMutableString *ms = [NSMutableString string];

    for (i = 0; i < CC_MD5_DIGEST_LENGTH; i++) {
        [ms appendFormat:@"%02x", (int)(digest[i])];
    }

    return [ms copy];
}
```

#### 4、进阶：自定义缓存策略的NSURLCache

思路如下：

- 1、网络请求开始时，首先会判断沙盒中是否存在缓存数据；[没有直接请求]
- 2、如果有，判断是否过期；没过期则使用本地数据。[过期则删除缓存，直接请求]
- 3、请求URL
代码如下：

```

// 判断是否为GET
-(NSCachedURLResponse *)cachedResponseForRequest:(NSURLRequest*)request{
    if ([request.HTTPMethod compare:@"GET"] != NSOrderedSame) {
        return [super cachedResponseForRequest:request];
    }
    return [self customResponseForRequest:request];
}

// 加密
-(NSString *)getFileName:(NSString *)url{
    return [url getMd5_32Bit];
}

-(NSString *)getOtherInfoName:(NSString *)url{
    return [[NSString stringWithFormat:@"%@-otherInfo",url] getMd5_32Bit];
}
- (NSString *)cacheFilePath:(NSString *)fileName{
    NSString *filePath = [NSString stringWithFormat:@"URLCache/%@",fileName];
    NSString *path = kFilePathAtCachWithName(filePath);
    NSFileManager *fileManager = [NSFileManager defaultManager];
    BOOL isDir;
    if([fileManager fileExistsAtPath:path isDirectory:&isDir] && isDir) {
    } else{
         [fileManager createDirectoryAtPath:path withIntermediateDirectories:YES attributes:nil error:nil];
    }
    return path;
}

-(NSCachedURLResponse*)customResponseForRequest:(NSURLRequest*)request{
    ​NSString *url = request.URL.absoluteString;//请求的链接
    
    // URL MD5 后作为缓存文件名。这里取出MD5后的文件名
    NSString*fileName = [self getFileName:url];
    // 缓存信息的文件名 @"%@-otherInfo",url
    NSString*otherInfoName = [self getOtherInfoName:url];
    // 创建/获取 缓存的目录路径
    NSString*filePath = [self cacheFilePath:fileName];
    // 缓存信息字典路径
    NSString*otherInfoPath = [self cacheFilePath:otherInfoName];
    
    ​NSFileManager *fileManger = [NSFileManager defaultManager];
    //当文件存在的时候
    if ([fileManger fileExistsAtPath:filePath]) {
        NSDate*date = [NSDate date];
        BOOL isOverdue = YES;
        NSDictionary *dicInfo = [NSDictionary dictionaryWithContentsOfFile:otherInfoPath];
        // 过期与否？
        if(self.catchTime > 0 && dicInfo) {
            NSDate*createDate = dicInfo[kTimeKey];
            double createTime = [date timeIntervalSinceDate:createDate];
            if (createTime < self.catchTime) {
                isOverdue = NO;
        }}
        //没有过期 加载缓存数据
        if (!isOverdue) {
            NSLog(@"访问缓存数据..");
            NSData *data = [NSData dataWithContentsOfFile:filePath];
            NSURLResponse *response = [[NSURLResponse alloc] initWithURL:request.URL MIMEType:dicInfo[kMIMETypeKey] expectedContentLength:data.lengthtextEncodingName:dicInfo[kTextEncodingNameKey]];
            NSCachedURLResponse *cacheResponse = [[NSCachedURLResponsealloc] initWithResponse:response data:data];
            return cacheResponse;
            
        }else{//已经过期，删除数据
            [fileManger removeItemAtPath :filePath error:nil];
            [fileManger removeItemAtPath :otherInfoPath error:nil];
        }
    }
    
    BOOL isExsit = [[self.dicResponse objectForKey:url] boolValue];
    if (!isExsit) {
        [self .dicResponse setObject:@(YES) forKey:url];
        NSLog(@"网络请求数据...");
        __block NSCachedURLResponse *cacheResponse = nil;
        [NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc] init] completionHandler:^(NSURLResponse*response, NSData *data, NSError *connectionError) {
            //移除请求记录
            [self.dicResponse removeObjectForKey:url];
            if (data && response) {
                NSMutableDictionary *dicInfo = [NSMutableDictionarydictionary];
                [dicInfo setObject :[NSDate date] forKey:kTimeKey];
                [dicInfo setObject :kUnNilStr(response.MIMEType) forKey:kMIMETypeKey];
                [dicInfo setObject :kUnNilStr(response.textEncodingName) forKey:kTextEncodingNameKey];
                BOOL state = [data writeToFile:filePath atomically:NO];
                NSLog(@"%d",state);
                [dicInfo writeToFile :otherInfoPath atomically:YES];
                cacheResponse = [[NSCachedURLResponse alloc] initWithResponse:response data:data];
            }
        }];
        return cacheResponse;
    }else{
        NSLog(@"该请求已存在");
    }
    return nil;
    
}

```
## 进阶篇

#### 1、UseProtocolCachePolicy - 默认缓存的机制

1、首先从apple提供的一个缓存策略的枚举说起：
```

typedef NS_ENUM(NSUInteger, NSURLRequestCachePolicy)
{
    // 默认值：按照HTTP协议进行缓存
    NSURLRequestUseProtocolCachePolicy = 0, 
    // 非缓存模式：不使用缓存数据，无论是否有本地缓存
    NSURLRequestReloadIgnoringLocalCacheData = 1,
    // 缓存模式：无论缓存是否过期都是用缓存，没有缓存就进行网络请求
    NSURLRequestReturnCacheDataElseLoad = 2,
    // 离线模式：无论缓存是否过期都是用缓存，没有缓存也不会进行网络请求
    NSURLRequestReturnCacheDataDontLoad = 3,

    NSURLRequestReloadIgnoringCacheData = NSURLRequestReloadIgnoringLocalCacheData,
    NSURLRequestReloadIgnoringLocalAndRemoteCacheData = 4, // Unimplemented
    NSURLRequestReloadRevalidatingCacheData = 5, // Unimplemented
};
```
这个枚举大家应该很熟悉，创建request时经常用到的。这里我进行了分类，主要是4类。其余3个排除。这里除了默认值，其他3个模式都很容易理解，下面以QQ空间的web页面拦截为例说明默认值是如何实现的：


2、拦截QQ WebView的请求
首先在webview上加载QQ空间页面，然后拦截webview上QQ空间登录的response，部分Header字段如下：
```

HTTP/1.1 200 OK
Content-Type    application/x-javascript
Server  QZHTTP-2.38.20
Content-Encoding    gzip
Cache-Control   public; max-age=86400
Proxy-Connection    Keep-alive
```
注意：这里的Cache-Control是我们的目标字段。

3、Cache-Control

这个字段用于指定该次整个请求中的缓存机制。常见的取值有public、private、no-cache、max-age、must-revalidate等，默认为private。QQ空间的Cache-Control解读如下：

- public : 所有内容都将被缓存(客户端和代理服务器都可缓存)；
- max-age=86400 : 缓存的内容将在86400秒后失效, 这个选项只在HTTP 1.1可用。


4、Cache位置

对于webview而言，缓存的数据默认在Library/Caches里。如下所示(ssh连接iphone)
```

# ls
Documents/  Library/  tmp/
# cd Library/ && ls
2222/  Caches/  Cookies/  Preferences/  WebKit/
# cd Caches/ && ls

Databases.db                         http_ui.ptlogin2.qq.com_0.localstorage
Snapshots/                           https_auth.alipay.com_0.localstorage
com.apple.WebKit.Networking/         https_passport.jd.com_0/
com.apple.WebKit.WebContent/         https_passport.jd.com_0.localstorage
http_h5.m.taobao.com_0.localstorage  
[截取部分缓存数据]
```


那在app收到response之后，会根据Cache-Control字段，选择缓存的内容以及缓存的时间，而这些缓存的机制则是由服务器端定义。


#### 2、缓存高清图片

经常会见到小图点开之后显示高清图片，那实现的一种方式就是:提前缓存高清图片,功能的实现要基于SDWebImage内部缓存的相关接口。

操作如下：拦截response，然后以url为key，数据为value，先缓存起来，在查看大图的时候，直接加载缓存的图片
代码如下：
```

- (void)storeCachedResponse:(NSCachedURLResponse *)cachedResponse forRequest:(NSURLRequest *)request {
    [super storeCachedResponse:cachedResponse forRequest:request];
    NSString *mimeType = cachedResponse.response.MIMEType;

    if ([mimeType hasPrefix:@"image"]) {
        NSString *url = request.URL.absoluteString;
        NSLog(@"cache url:%@", url);

        SDImageCache *cache = [SDImageCache sharedImageCache];
        [cache storeImage:[UIImage imageWithData:cachedResponse.data] forKey:url toDisk:NO];
    }

}
```
但是注意，这种方法需要URL固定，如果Referer到其他站就会有问题了。

最佳的方案是使用NSURLProtocol解决。详细的可以参考我的NSURLProtocol博文。
如下：

```

- (void)startLoading {
    NSURLSession *session = [NSURLSession sharedSession];
    NSMutableURLRequest *request = [self.request mutableCopy];
    [NSURLProtocol setProperty:@(YES) forKey:kProtocolHandledKey inRequest:request];
    __weak typeof(self) weakSelf = self;
    self.sessionTask = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        if (error) {
            return;
        }
        NSString *mimeType = response.MIMEType;
        if ([mimeType hasPrefix:@"image"]) {
            NSString *url = weakSelf.request.URL.absoluteString;
            SDImageCache *cache = [SDImageCache sharedImageCache];
            [cache storeImage:[UIImage imageWithData:data] forKey:url toDisk:NO];
        }
        [weakSelf.client URLProtocol:weakSelf didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageAllowedInMemoryOnly];
        [weakSelf.client URLProtocol:weakSelf didLoadData:data];
        [weakSelf.client URLProtocolDidFinishLoading:weakSelf];
    }];
    [self.task resume];
}
- (void)stopLoading {
    [self.sessionTask cancel];
    self.sessionTask = nil;
}

```

我们使用了自定义的NSURLProtocol对所有http(s)请求进行了拦截，并在资源加载完成的时候进行对mime-type的判断，如果是image，那我们就以请求的URL为key，把请求得到的数据转换为UIImage存入SDImageCache中。



## Http Cache机制

需求：服务器的文件存贮，大多不是一层不变的，往往在资源变动后就重新生成一个链接。但如果本地已经有了缓存并且在时效期内。那如何即时更新缓存？这种情况下需要借助 ETag 或 Last-Modified 等判断本地缓存有效性。

下边先解说理论，然后实战。


Http的Cache机制总共有4个组成部分：
Last-Modified（If-Modified-Since）、Etag（If-None-Match）、Cache-Control、Expires


#### 1、Last-Modified / If-Modified-Since
Last-Modified是响应头，If-Modified-Since是请求头。Last-Modified把Web资源的最后修改时间告诉客户端，客户端在下次请求此Web资源的时候，会把上次服务端响应的最后修改时间作为If-Modified-Since
的值发送给服务器，服务器可以通过这个值来判断是否需要重新发送，如果不需要，就简单的发送一个
304状态码，客户端将从缓存里直接读取所需的Web资源。如果有更新，返回HTTP 200和更新的页面内容，
并且携带新的”ETag”和”LastModified”。使用这个机制，能够避免重复发送文件给浏览器，不过仍然会产生一个HTTP请求。

#### 2、ETag / If-None-Match

ETag是响应头，If-None-Match是请求头。Last-Modified / If-Modified-Since的主要缺点就是它只能精确到秒的级别，一旦在一秒的时间里出现了多次修改，那么Last-Modified / If-Modified-Since是无法体现的。相比较，ETag / If-None-Match没有使用时间作为判断标准，而是使用一个特征串。Etag把Web资源的特征串告诉客户端，客户端在下次请求此Web资源的时候，会把上次服务端响应的特征串作为If-None-Match的值发送给服务端，服务端可以通过这个值来判断是否需要从重新发送，如果不需要，就简单的发送一个304状态码，客户端将从缓存里直接读取所需的Web资源。因此，HTTP/1.1利用Entity Tag头提供了更加严格的验证。
 
####  3、Expires  && Cache-Control
当服务器发出响应的时候，可以通过两种方式来告诉客户端缓存请求：

第一种是Expires，比如：Expires: Mon, 26 Dec 2015 02:26:11 GMT在此日期之前，客户端都会认为缓存是有效的。
不过Expires有缺点，比如说，服务端和客户端的时间设置可能不同，这就会使缓存的失效可能并不能精确的按服务器的预期进行。

第二种是Cache-Control，比如：Cache-Control: max-age=3600
这里声明的是一个相对的秒数，表示从现在起，3600秒内缓存都是有效的，这样就避免了服务端和客户端时间不一致的问题。

但是Cache-Control是HTTP1.1才有的，不适用与HTTP1.0，而Expires既适用于HTTP1.0，也适用于HTTP1.1，所以说在大多数情况下同时发送这两个头会是一个更好的选择，当客户端两种头都能解析的时候，会优先使用Cache-Control基础知识

#### 4、实战 etag的使用

如下代码所示：

```

 AppDelegate.m
 1、 设置网络缓存
NSURLCache *cathe = [[NSURLCache alloc] initWithMemoryCapacity:4*1024*1024 diskCapacity:20*1024*1024 diskPath:nil];
[NSURLCache setSharedURLCache:cathe];

2、
ViewController.m
 //服务器返回的etag
@property (nonatomic, copy) NSString *etag;

 /** 原理说明
     1. 缓存策略：NSURLRequestReloadIgnoringCacheData，忽略本地缓存。
     2. 服务器响应结束后，要记录 Etag;
     3. 在发送请求时，设置 If-None-Match，传入etag
     4. 连接结束后，要判断响应头的状态码，如果是304，说明本地缓存内容没有发生变化
*/

- (void)getData {
        NSURL *url = [NSURL URLWithString:@"GET"];
        NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url cachePolicy:NSURLRequestReloadIgnoringCacheData timeoutInterval:10.0];

        //设置请求头，如果etag length不为0，说明已经有缓存了
        if (self.etag.length > 0) {
            NSLog(@"设置了etag: %@", self.etag);
            [request setValue:self.etag forHTTPHeaderField:@"IF-None-Match"];
        }
        /**
         *Etag = "\"6a0a9-918a2d800bd40\"";
         *可以在请求中增加一个etag跟服务器返回的etag进行对比
         *就能够判断服务器对应的资源是否发生变化，具体更新的时间，由request自行处理
         */
        [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError)
        {
            NSHTTPURLResponse *httpResponse = (NSHTTPURLResponse *)response;
            NSLog(@"%@ %@", httpResponse.allHeaderFields, httpResponse);
            /**
             *如果服务器的状态码是304，说明数据已经被缓存，服务器不再需要返回数据
             *需要从本地缓存获取被缓存的数据
             */
            if (httpResponse.statusCode == 304) {
                NSLog(@"加载本地数据");
                NSCachedURLResponse *cachedResponse = [[NSURLCache sharedURLCache] cachedResponseForRequest:request];
                data = cacheResponse.data;
            }

            //记录etag
            self.etag = httpResponse.allHeaderFields[@"etag"];

        }];

```


相关参考：
[https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)













