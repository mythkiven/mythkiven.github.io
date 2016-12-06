---
layout: post
title:  "WKWebView使用小结"
categories: WKWebView
tags: WKWebView
author: 3行代码
---

* content
{:toc}

## 前言

一直使用UIWebView，使用久了就会发现，相比 Nirtro JavaScript引擎的Safari，WebView实在太过笨重，还会有内存泄漏。
然而自诩有60fps刷新率、内置手势、高效的app和web信息交换通道、和 Safari相同的JavaScript引擎，WKWebView明显的优化了这些不足，下边就个人使用体会做个小结。

## 1、基础

WKWebView是在iOS8中新增的，如果需要兼容iOS7，需要判断系统版本，然后选择性的使用。

```
#import <WebKit/WebKit.h>
WKWebView *webView = [[WKWebView alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
// 加载过程代理
webView.navigationDelegate = self;
// UI界面相关代理，原生控件支持。
webView.UIDelegate = self;
// 设置是否透明，默认yes，不透明的。设置为NO，可以在web下面放一个img
webView.opaque = NO;   
// web内手势左右滑动导航
webView.allowsBackForwardNavigationGestures =YES;  
// 设置加载进度：自带的进度条: webView.estimatedProgress
// get是否加载 webView.loading
// get标题 webView.title
[webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.3code.info"]]];
```

#### 1.1 初始化

// 初始化方法，两种

    // 默认初始化
    - (instancetype)initWithFrame:(CGRect)frame;
    // 根据对webview的相关配置，进行初始化
    - (instancetype)initWithFrame:(CGRect)frame configuration:(WKWebViewConfiguration *)configuration NS_DESIGNATED_INITIALIZER;
    
    使用见下文的JS交互


#### 1.2 监听title和estimatedProgress

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
    else
        if ([keyPath isEqualToString:@"title"] && object == self.webView)
        {
            NSString * titleText = self.webView.title;
            title.text = titleText; 
        }
        else {
            [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
}

注意：移除掉观察者
 [self.webView removeObserver:self forKeyPath:@"title"];
 [self.webView removeObserver:self forKeyPath:@"estimatedProgress"];

```

#### 1.3 加载页面

1、加载网页与HTML代码的方式与UIWebView相同，代码如下：
```
[self.webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:@"http://www.3code.info"]]];
```

2、不同的是，WKWebView不支持用loadRequest的方法加载本地的静态HTML，可以用

    [self.webView loadFileURL:[NSURLfileURLWithPath:url] allowingReadAccessToURL:[NSURL fileURLWithPath:accessPath]];
其中的前面一个参数url是指想要加载的具体哪个文件(比如指向一个index.html)，后面一个参数它是指系统能够访问的文件路径，就是HTML所需的相关JS、CSS 等文件所在的目录。
注意的是，要放进沙盒中。

#### 1.4 代理方法

```

/* WKNavigationDelegate  */

//1、在发送请求之前，决定是否跳转
- (void)webView: decidePolicyForNavigationAction: 

//2、页面开始加载时调用
- (void)webView: didStartProvisionalNavigation: 
//3、当内容开始返回时调用
- (void)webView: didCommitNavigation: 
//4、页面加载完成之后调用
- (void)webView: didFinishNavigation: 
// 页面加载失败时调用
- (void)webView: didFailProvisionalNavigation: 

/* 决定是否执行页面跳转的代理方法 */
// 接收到服务器跳转请求之后调用
- (void)webView:  didReceiveServerRedirectForProvisionalNavigation: 
// 当客户端收到服务器的响应头，根据response相关信息，可以决定这次跳转是否可以继续进行
1、需要执行对应的block 2、可以拦截自定义的URL来实现JS调用OC方法
- (void)webView: decidePolicyForNavigationResponse: 
// 根据webView、navigationAction相关信息决定这次跳转是否可以继续进行
- (void)webView: decidePolicyForNavigationAction: 

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


#### 2.5 缓存

但是他有一个最致命的缺陷，就是WKWebView的请求不能被NSURLProtocol截获。而我们团队开发的app中对于H5容器最佳的优化点主要就在于使用NSURLProtocol技术对于H5进行离线包的处理和H5的图片和Native的图片公用一套缓存的技术。因为该问题的存在，目前我们团队还没有使用WKWebView代替UIWebVIew。

在UIWebView，使用NSURLCache缓存，通过setSharedURLCache可以设置成我们自己的缓存，但WKWebView不支持NSURLCache。所以涉及到离线缓存的问题时，要注意区分使用哪个。

#### 2.6 类

WKBackForwardList: 之前访问过的 web 页面的列表，可以通过后退和前进动作来访问到。
WKBackForwardListItem: webview 中后退列表里的某一个网页。
WKFrameInfo: 包含一个网页的布局信息。
WKNavigation: 包含一个网页的加载进度信息。
WKNavigationAction: 包含可能让网页导航变化的信息，用于判断是否做出导航变化。
WKNavigationResponse: 包含可能让网页导航变化的返回内容信息，用于判断是否做出导航变化。
WKPreferences: 概括一个 webview 的偏好设置。
WKProcessPool: 表示一个 web 内容加载池。
WKScriptMessage: 包含网页发出的信息。
...


## 3、WKWebView的常见问题

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

 