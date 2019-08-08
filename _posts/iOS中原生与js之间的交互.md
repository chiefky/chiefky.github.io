##iOS中原生与js之间的交互
native与js的交互分为两种分别是：JS调原生和原生调JS，JS调用原生是指通过JS唤醒APP原生的逻辑或者功能；原生调JS是指，通过原生组件直接调用JS中的方法。
下面简单列举一下两种交互的几种实现方式：
###JS调用原生
#####一、拦截跳转（基于UIWebView）
是我们最常见的一种方式，也是最简单，最容易理解的一种。我们可以在UIWebView的代理方法中拦截每一个请求，如果是特殊的链接就可以做一些事情，比如跳转、执行某些方法等。示例如下：

``` 
// JS端
window.location = 'app://login?account=13011112222&password=123456';

// iOS端
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {  
  NSString *scheme = request.URL.scheme;
  NSString *host = request.URL.host;
  
  // 一般用作交互的链接都会有一个固定的协议头，这里我们一“app”作为协议头为了，实际项目中可以修改
  if ([scheme isEqualToString:@"app"]) { // scheme为“app”说明是做交互的链接
      if ([host isEqualToString:@"login"]) { // host为“login”对应的就是登录操作
          NSDictionary *paramsDict = [request.URL getURLParams];
          NSString *account = paramsDict[@"account"];
          NSString *password = paramsDict[@"password"];
          NSString *msg = [NSString stringWithFormat:@"执行登录操作，账号为：%@，密码为：%@", account, password];
          UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"原生弹窗" message:msg delegate:nil cancelButtonTitle:@"好" otherButtonTitles:nil, nil];
          [alert show];
      }
  return YES;
}
```
####二、拦截跳转（基于WKWebView）
类似UIWebView，在WK中我们同样可以拦截跳转，原理相同，代码不同。示例如下：

```
// JS端
window.location = 'app://login?account=13011112222&password=123456';

// iOS端
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
  NSURLRequest *request = navigationAction.request;
  NSString *scheme = request.URL.scheme;
  NSString *host = request.URL.host;
  
  // 一般用作交互的链接都会有一个固定的协议头，这里我们一“app”作为协议头为了，实际项目中可以修改
  if ([scheme isEqualToString:@"app"]) { // scheme为“app”说明是做交互的链接
      if ([host isEqualToString:@"login"]) { // host为“login”对应的就是登录操作
          NSDictionary *paramsDict = [request.URL getURLParams];
          NSString *account = paramsDict[@"account"];
          NSString *password = paramsDict[@"password"];
          NSString *msg = [NSString stringWithFormat:@"执行登录操作，账号为：%@，密码为：%@", account, password];
          UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"原生弹窗" message:msg delegate:nil cancelButtonTitle:@"好" otherButtonTitles:nil, nil];
          [alert show];
      }
      
      // ... 这里可以继续加 else if
      
      decisionHandler(WKNavigationActionPolicyCancel);
      return;
  }
  decisionHandler(WKNavigationActionPolicyAllow);
}

```

####三、JSContext（基于UIWebView）

JSContext即JavaScriptContext，这个东西在UIWebView中可以拿到，但是在WKWebView中却是取不到了，所以只能用在UIWebView中。

JSContext的原理就是iOS暴露出去一个遵守<JSExport>协议的对象给JS，JS可以直接调用该对象的public方法。

```

// JS端，app是iOS中注册的一个对象
app.login("13011112222", "123456");

// iOS端
// 每次嵌入页面加载完毕都要给jsContext赋值，否则在js端调用可能会失效。
- (void)webViewDidFinishLoad:(UIWebView *)webView {
  self.jsContext = [webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
  self.jsContext.exceptionHandler = ^(JSContext *context, JSValue *exception) {
      context.exception = exception;
      NSLog(@"异常信息：%@", exception);
  };
  // app是随便取的名字，可以改，改了之后JS要同步修改。如果Android端使用@JavaScriptInterface的形式，那么还要保证Android、iOS两端同步，建议都用app
  self.jsContext[@"app"] = [[JSContextModel alloc] init];
}

// JSContextModel，
@protocol JsContextExport<JSExport>
/**
 * 登出方法，js调用的方法名也是logout
 */
- (void)logout;
/**
 * 登录方法，JSExportAs的作用就是给OC方法导出一个js方法名，例如下面的方法js调用就是 login("your account", "your password")。在多参数的方法声明时必须使用这种方式
 */
JSExportAs(login, - (void)loginWithAccount:(NSString *)account   password:(NSString *)password);
/**
 * 获取登录信息
 * @return 当前登录用户的身份信息。JSContext方式调用OC时，方法的返回值只能是NSString、NSArray、NSDictionary、NSNumber、BooL，其他类型不能解析
 */
- (NSDictionary *)getLoginUser;
@end

@interface JSContextModel : NSObject<JsContextExport>
@end

```
#### 四、WebKit（基于WKWebView）
window.webkit.messagehandlers.<name>.postMessage是apple推荐使用的WKWebView的JS交互方式，使用起来比较简单，不支持callback回调。

```

// js
window.webkit.messageHandlers.login.postMessage({
  'account': '13000000000',
  'password': '123456'
});

// iOS - 初始化WKWebView时设置 configuration
self.webView = [[WKWebView alloc] initWithFrame:CGRectZero configuration:[[WKWebViewConfiguration alloc] init]];
WKUserContentController *confVc = self.webView.configuration.userContentController;
  [confVc addScriptMessageHandler:self name:@"login"];
  
// iOS - 在ScriptMessageHandler 的代理方法中处理
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
  if ([message.name isEqualToString:@"login"]) {
      if (![message.body isKindOfClass:[NSDictionary class]]) {
          return;
      }
      NSDictionary *data = message.body;
      NSString *account = data[@"account"];
      NSString *password = data[@"password"];
      
      NSString *msg = [NSString stringWithFormat:@"执行登录操作，账号为：%@，密码为：%@", account, password];
      UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"原生弹窗" message:msg delegate:nil cancelButtonTitle:@"好" otherButtonTitles:nil, nil];
      [alert show];
      return;
  }
}

``` 

####五、JsBridge（适用两种组件）

UIWebView、WKWebView同时支持，且方法名完全没有差异，但是特别要注意的一点就是：iOS原生是不支持这种方式的，我们需要依赖于一个三方库 —— WebViewJavascriptBridge，这是一个很有名的库，具体有多牛逼这里也不做过多需求，百度一下你就知道。

```
// 首先调用JSBridge初始化代码，完成后再设置其他
initJsBridge(function () {
  $("#getOS").click(function () {
        // 通过JsBridge调用原生方法，写法固定，第一个参数时方法名，第二个参数时传入参数，第三个参数时响应回调
        window.WebViewJavascriptBridge.callHandler('getOS', null, function (response) {
        showResponse(response);
      });
  });
 });
``` 

###原生调js
####一、UIWebView直接调用Js方法，示例代码如下：
```
[self.webView stringByEvaluatingJavaScriptFromString:@"showResponse('123')"];
```

####二、WKWebView直接调用Js方法，示例代码如下：
```
[self.webView evaluateJavaScript:@"showResponse('点击了原生的按钮22222222222')" completionHandler:^(id _Nullable response, NSError * _Nullable error) {
      if (error) {
          NSLog(@"%@", error);
      } else {
          NSLog(@"%@", response);
      }
 }];
```
它相对于UIWebView而言最大的优点就是支持callback，不想UIWebView那样只能一去不复返。

##### 补：如果是使用了JsBridge，对应是原生调JS实现代码：

```
[self.bridge callHandler:@"jsbridge_getJsMessage" data:@"点击了原生的按钮222222222" responseCallback:^(id responseData) {
  UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"显示jsbridge返回值" message:responseData delegate:nil cancelButtonTitle:@"好" otherButtonTitles:nil, nil];
   [alert show];
}];

```