---
layout: post
title:  "WKWebView使用小结"
categories: WKWebView
tags: WKWebView、Native与JS交互、离线缓存与NSURLProtocol
author: 3行代码
---

* content
{:toc}

## 前言

WKWebView出来有段时间了，项目中一直在使用UIWebView因为要兼容IOS7，前段时间产品那边忽然想开了，ignoreIOS7。于是我们开始使用WkWebView，下边总结了前段时间学习的笔记，记录下来。如果需要demo的还请留言。
早在2014年的WWDC大会上就了解过WebKit，自诩有60fps刷新率、内置手势、和Safari相同的JavaScript引擎等等众多优势，相比之下UIWebview显得比较low。

就这段时间的使用体验来看，变化集中在：
- 1、内存消耗少；
- 2、协议接口更细化，更丰富；
- 3、有导航的概览、网站窗口、网站列表等概念；


## 1、基础

基本属性

```
#import <WebKit/WebKit.h>
WKWebView *webView = [[WKWebView alloc] initWithFrame:[[UIScreen mainScreen] bounds]];

// 代理
webView.navigationDelegate = self;
webView.UIDelegate = self;

// opaque不透明的。
webView.opaque = NO;  

// YES证明webView还在加载数据,所有数据加载完毕后,webView就会为No
// webView.loading;

// web内手势左右滑动导航
webView.allowsBackForwardNavigationGestures =YES;

[webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.3code.info"]]];
```

#### 1.1 初始化

初始化方法，两种：

    // 默认初始化
    - (instancetype)initWithFrame:(CGRect)frame;
    // 根据对webview的相关配置，进行初始化
    - (instancetype)initWithFrame:(CGRect)frame configuration:(WKWebViewConfiguration *)configuration NS_DESIGNATED_INITIALIZER;
     


#### 1.2 加载页面

// 通过一个网页URL来加载一个WKWebView
-loadRequest: 
// 根据一个文件,加载一个WKWebView，后边是资源路径 【IOS9】
-loadFileURL: allowingReadAccessToURL: 
// 将html文件读取为字符串从而加载为WKWebView，baseURL是我们自己设置的资源路径
-loadHTMLString: baseURL: 
// 下边方法使用的比较少，但更加自由，其中data是文件数据，MIMEType是文件类型，characterEncodingName是编码类型，baseURL是素材资源路径
-(nullable WKNavigation *)loadData:(NSData *)data MIMEType:(NSString *)MIMEType characterEncodingName:(NSString *)characterEncodingName baseURL:(NSURL *)baseURL NS_AVAILABLE(10_11, 9_0);


#### 1.3 协议方法

##### 1.3.1  需要注意的几个协议方法：

1、-(void)webView: decidePolicyForNavigationAction: 

注意的是：
A、需要call decisionHandler，允许或者不允许加载，否则报错。
B、
当点击link时，如果有标签_blank,在webview中不会新开窗口，但是在WKWeb中，可以控制新开一个WKWebview。这时可以通过该方法的第一个参数WKNavigationAction的属性，来判断是否需要开新的view:

    if(!navigationAction.targetFrame.mainFrame){开新窗口}
这时就会调用 createWebViewWithConfiguration方法return的wkwebview，来加载新的页面。但如果没有实现这个协议，那么操作link就不会有反应了。
我在开发中实际没有用到需要开view的复杂情况，所以我的处理也是简单粗暴，直接处理掉所有的_blank标签。

2、-(void) webView: decidePolicyForNavigationResponse: 
这个也需要call decisionHandler,允许or不允许加载。

##### 1.3.2 其它协议方法汇总

```

/* WKNavigationDelegate  */

//1、【即将发送request请求】 可以决定是否请求，拦截request、cookie等。call decisionHandler
- (void)webView: decidePolicyForNavigationAction: 
//2、【开始请求页面】
- (void)webView: didStartProvisionalNavigation: 
//3、【收到response】 根据response决定要不要继续加载。call decisionHandler
-(void) webView: decidePolicyForNavigationResponse: 
//4、【收到web内容，开始渲染】 可以注入JS
- (void)webView: didCommitNavigation: 
//5、【页面渲染完成】可以注入JS
- (void)webView: didFinishNavigation: 

// 【网页内容被终止】
- (void)webViewWebContentProcessDidTerminate:
// 【错误：页面跳转失败】
- (void)webView: didFailNavigation: 
// 【错误：启动加载数据失败】
- (void)webView: didFailProvisionalNavigation: 
// 【页面校验身份】
- (void)webView: didReceiveAuthenticationChallenge: 
// 【重定向】
- (void)webView: didReceiveServerRedirectForProvisionalNavigation
 

/* UIDelegate  */
当页面中有调用了js的alert、confirm、prompt方法就会触发以下方法
// 页面中有输入框弹出警告框时调用
- (void)webView: runJavaScriptTextInputPanelWithPrompt: 
// 页面中有确认框弹出警告框时调用
- (void)webView: runJavaScriptConfirmPanelWithMessage:( 
// 警告框页面中有警告框弹出警告框时调用
- (void)webView: runJavaScriptAlertPanelWithMessage: 

```

#### 1.5 导航刷新相关

网页导航刷新相关函数和UIWebview几乎一样，不同的是增加了函数reloadFromOrigin和goToBackForwardListItem。

//会比较网络数据是否有变化，没有变化则使用缓存，否则从新请求。
-(WKNavigation *)reloadFromOrigin; 

//比向前向后更强大，可以跳转到某个指定历史页面
-(WKNavigation *)goToBackForwardListItem:(WKBackForwardListItem *)item; 

#### 1.6 KVO监听标题和进度条

可以利用title和estimatedProgress，用KVO去监听，实时改变进度和标题:
```
[self.webView addObserver:self forKeyPath:@"title" options:NSKeyValueObservingOptionNew context:NULL];
[self.webView addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:NULL];

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if ([keyPath isEqualToString:@"estimatedProgress"] && object == self.webView) {
        [self.progress setAlpha:1.0f];
        [self.progress setProgress:self.webView.estimatedProgress animated:YES];
        
        if(self.webView.estimatedProgress >= 1.0f) {
            [UIView animateWithDuration:0.3 delay:0.3 options:UIViewAnimationOptionCurveEaseOut animations:^{
                [self.progress setAlpha:0.0f];
            } completion:^(BOOL finished) {
                [self.progress setProgress:0.0f animated:NO];
            }];
        }
    }
    else if ([keyPath isEqualToString:@"title"] && object == self.webView) {
            NSString * titleText = self.webView.title;
            self.title = titleText; 
    } else {
            [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
}

注意：移除掉观察者
 [self.webView removeObserver:self forKeyPath:@"title"];
 [self.webView removeObserver:self forKeyPath:@"estimatedProgress"];

```


##  2、Native与JS的通信

几点注意：

1、线程方面需要注意的是:
- iframe调用Navite是异步执行的(android中是同步的)；
- stringByEvaluatingJavaScriptFromString是同步执行的；

2、编码方面：
- 通信过程中，从JS向Native传递可能会出现乱码情况，最好进行Base64编码然后附加在url后面进行传递
- Native调用JS需要在页面加载完成之后进行；
- 键盘的控制不好处理: [obj becomeFirstResponder]方法不可用

#### 2.1 iOS6:JS与Native的通信：iFrame

iOS6原生没有提供js直接调用Objective-C的方式，只能通过UIWebView的UIWebViewDelegate协议拦截iframe请求实现。

1、Native调用JS  stringByEvaluatingJavaScriptFromString: 

如果需要OC给JS回调，也用这个方法

    NSString *jsString = @" var p = document.createElement('p'); \
                            p.innerText = 'New page';            \
                            document.body.appendChild(p);        \
    ";
    [_webView stringByEvaluatingJavaScriptFromString:jsString];

2、JS调用Native  webView:shouldStartLoadWithRequest:

JS中：
```
    function loadURL(url) {
        var iFrame;
        iFrame = document.createElement("iframe");
        iFrame.setAttribute("src", url);
        iFrame.setAttribute("style", "display:none;");
        iFrame.setAttribute("height", "0px");
        iFrame.setAttribute("width", "0px");
        iFrame.setAttribute("frameborder", "0");
        document.body.appendChild(iFrame);
        // 发起请求后这个iFrame就没用了，从dom上移除掉
        iFrame.parentNode.removeChild(iFrame);
        iFrame = null;
    }
比如我们在js代码中，调用js方法：
    function iOS_alert() {//调用自定义对话框
        //window.location.href调用的话会导致前一次操作取消
        loadURL("alert://baidu");
    }
    // 以下是为了解决OC回调给JS时候，可能存在的阻塞主线程情形。可以结合实际决定要不要使用。
    function asyncAlert(content) {
    setTimeout(function(){
               alert(content);
               },1);
    }
    function setLocation(location) {
        asyncAlert(location);
        document.getElementById("Value").value = location;
    }

```

Native中：

```
// url.scheme 区分是不是我要拦截的，url.host决定调用哪个方法。
//return YES则会加载URL，NO不加载URL
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
    NSURL * url = [request URL];
    if ([[url scheme] isEqualToString:@"alert"]) {//拦截alert
        UIAlertView * alertView = [[UIAlertView alloc] initWithTitle:@"test" message:[url host] delegate:nil cancelButtonTitle:@"OK" otherButtonTitles:nil];
        [alertView show];
        return NO;
    }
    return YES;
}
```

#### 2.2 iOS7:JS与Native的通信：JavaScriptCore

iOS7中加入了JavaScriptCore.framework框架。把WebKit的JavaScript引擎用OC 封装。该框架让Objective-C和JavaScript代码直接的交互变得更加的简单。

1、JS调用Native
JS中：
```
<button type="button" onclick="showAlert('YES')">Click</button>
function showAlert(msg){
            alert(msg);
        }
```
Native中：
```
JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];

- (void)webViewDidFinishLoad:(UIWebView *)webView {
    JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    //定义好JS要调用的方法, alert就是方法名
    context[@"alert"] = ^() {
        NSArray *args = [JSContext currentArguments];
        UIAlertView *alertView = [[UIAlertView alloc] init
        [alertView show];
        
        for (JSValue *jsVal in args) {
            NSLog(@"%@", jsVal.toString);
        }
    };
}

```

2、OC调用JS

方法1：stringByEvaluatingJavaScriptFromString:不过该方法是同步执行的，可能会阻塞UI的刷新。

方法2： -evaluateScript

    JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    NSString *textJS = @"showAlert('弹出message')";
    [context evaluateScript:textJS];
方法3： -callWithArguments

    JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    [context[@"payResult"] callWithArguments:@[@"支付结果"]];

需要注意的是:WKWebView不支持通过如下的KVC的方式创建JSContext：

    JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];

在WKWebView中有OC和JS交互的方式，更简洁，因此JavaScriptCore对我来说是昙花一现，基本不使用的。


####  2.3 iOS8:JS与Native的通信：WKScriptMessageHandler

1、JS调用Native：

注入JS
```
// 除了和UIWebView加载一个隐藏的ifame之外，WKWebView自身还提供了一套js调用native的方法,如下：

// 创建配置
WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
// 提供javaScript向webView发送消息的方法
 WKUserContentController *userContent = [[WKUserContentController alloc] init];
// 注入JS对象：NativeMethod，注意：self需要遵守WKScriptMessageHandler协议，结束时需要移除
[userContent addScriptMessageHandler:self name:@"NativeMethod"];
// 将UserContentController设置到配置文件中
config.userContentController = userContent;
// 自定义配置创建WKWebView
WKWebView *webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:config];
```

Native拦截JS
```
- (void)userContentController:(WKUserContentController *)userContentController
      didReceiveScriptMessage:(WKScriptMessage *)message {
    if ([message.name isEqualToString:@"NativeMethod"]) {
        NSLog(@"message.body:%@", message.body);
        //如果是自己定义的协议, 再截取协议中的方法和参数, 判断无误后在这里手动调用oc方法
        NSMutableDictionary *param = [self queryStringToDictionary:message.body];
        NSLog(@"get param:%@",[param description]);
        
        NSString *func = [param objectForKey:@"func"];
        
        //调用本地函数
        if([func isEqualToString:@"alert"])
        {
            [self showMessage:@"来自网页的提示" message:[param objectForKey:@"message"]];
        }
     
    }
}
```
// 注意decisionHandler必须执行。

2、Native调用JS：

    - (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^)(id, NSError *))completionHandler;
    //使用，同上webview
    NSString *jsString = @" var p = document.createElement('p'); \
                            p.innerText = 'New Line';            \
                            document.body.appendChild(p);        \
    ";
    [_wkView evaluateJavaScript:jsString completionHandler:^(id item, NSError *error) {
    // 这里是回调的结果
    }];

//completionHandler 拥有两个参数，一个是返回错误，一个可以返回执行脚本后的返回值，

#### 2.4 当然也可以使用三方库 进行通信

## 3、 缓存与NSURLProtocol

WKWebView的请求不能被NSURLProtocol截获。而我们团队开发的app中对于H5容器最佳的优化点主要就在于使用NSURLProtocol技术对于H5进行离线包的处理和H5的图片和Native的图片公用一套缓存的技术。因为该问题的存在，目前我们团队还没有使用WKWebView代替UIWebVIew。

在UIWebView，使用NSURLCache缓存，通过setSharedURLCache可以设置成我们自己的缓存，但WKWebView不支持NSURLCache。所以涉及到离线缓存的问题时，要注意区分使用哪个。
 


## 4、WKWebView的常见问题

崩溃问题: Terminating app due to uncaught exception ‘NSInternalInconsistencyException’ reason: ‘Completion handler passed to - [ViewController webView: decidePolicyForNavigationAction: decisionHandler:] was not called’
解决办法: 在webView:decidePolicyForNavigationAction:decisionHandler:函数里需执行decisionHandler的block
```

- (void)webView:(WKWebView*)webView decidePolicyForNavigationAction:(WKNavigationAction*)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler
{
    if (navigationAction.navigationType == WKNavigationTypeLinkActivated) {
        decisionHandler(WKNavigationActionPolicyAllow);
    }
    else {
        decisionHandler(WKNavigationActionPolicyCancel);
    }
}
```

#### post请求

在Native中使用WKWeb发送POST暂时不支持的，需要通过JS走：

```
<script>
         //调用格式： post('URL', {"key": "value"});
         function post(path, params) {
             var method = "post";
             var form = document.createElement("form");
             form.setAttribute("method", method);
             form.setAttribute("action", path);

             for(var key in params) {
                 if(params.hasOwnProperty(key)) {
                     var hiddenField = document.createElement("input");
                     hiddenField.setAttribute("type", "hidden");
                     hiddenField.setAttribute("name", key);
                     hiddenField.setAttribute("value", params[key]);

                     form.appendChild(hiddenField);
                 }
             }
             document.body.appendChild(form);
             form.submit();
         }
     </script>
```
然后在- (void)webView: didFinishNavigation: 中调用如下代码：
```
- (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation {
     if (self.ONCE) {
         // 调用使用JS发送POST请求的方法
         [self postRequestWithJS];
         // 将Flag置为NO（后面就不需要加载了）
         self.needLoadJSPOST = NO;
     }
 }
 // 调用JS发送POST请求
 - (void)postRequestWithJS {
     // 发送POST的参数
     NSString *postData = @"\"username\":\"aaa\",\"password\":\"123\"";
     // 请求的页面地址
     NSString *urlStr = @"http://www.baidu.com";
     // 拼装成调用JavaScript的字符串
     NSString *jscript = [NSString stringWithFormat:@"post('%@', /\%\}/@});", urlStr, postData];
     // NSLog(@"Javascript: %@", jscript);
     // 调用JS代码
     [self.webView evaluateJavaScript:jscript completionHandler:^(id object, NSError * _Nullable error) {

     }];
 }
```

## 4、其他

 iOS Xcode工程目录的 folder 和 group的区别(蓝色和黄色文件夹的区别)：

 group 一般只在你的工程中是文件夹的形式，但是在本地的目录中还是以散乱的形式放在一起的，除非你是从外部以group的形式引用进来的。

folder 只能作为资源，整个引用进项目，不能编译代码，也就是说，以folder形式引用进来的文件，不能被放在complie sources列表里面。

从外部拖入文件时，
选择Creat Groups：文件夹黄色的   
Creat Floder：这里我选择的是以folder的形式引用文件夹，点击完成，如下图所示，文件夹是蓝色的

 