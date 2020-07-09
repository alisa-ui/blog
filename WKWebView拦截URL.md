### WKWebView拦截URL

`WKWebView`与`UIWebView`拦截`URL`的处理方式基本一样。除了代理方法和`WKWebView`的使用不太一样。

##### 加载WKWebView



```objectivec
- (void)loadWKWebView {
    WKWebViewConfiguration *config = [WKWebViewConfiguration new];
    WKPreferences *preferences = [WKPreferences new];
    preferences.minimumFontSize = 30;
    config.preferences = preferences;
    self.webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:config];
    self.webView.UIDelegate = self;
    self.webView.navigationDelegate = self;
    
    NSURL *URL = [[NSBundle mainBundle] URLForResource:@"index" withExtension:@"html"];
    NSURLRequest *request = [[NSURLRequest alloc] initWithURL:URL];
    [self.webView loadRequest:request];
    [self.view addSubview:self.webView];
}
```

##### 拦截URL

使用`WKNavigationDelegate`中的代理方法，拦截自定义的`URL`来实现`JS`调用`OC`方法。注意调用`decisionHandler`这个`block`，否则会导致`App`崩溃。



```objectivec
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler {
    NSString *scheme = navigationAction.request.URL.scheme;
    if ([scheme isEqualToString:@"test"]) {
        [self handleURL:navigationAction.request.URL];
        decisionHandler(WKNavigationActionPolicyCancel); //不允许跳转
        return;
    }
    decisionHandler(WKNavigationActionPolicyAllow); // 允许跳转
}
```



```objectivec
- (void)handleURL:(NSURL *)URL {
    if ([URL.host isEqualToString:@"showAlert"]) {
        NSString *jsStr = [NSString stringWithFormat:@"alertWithMessage('%@')",@"OC调用JS的方法"];
        [self.webView evaluateJavaScript:jsStr completionHandler:^(id _Nullable result, NSError * _Nullable error) {
            NSLog(@"%@----%@",result, error);
        }];
    }
    
    else if ([URL.host isEqualToString:@"setColor"]) {
        NSArray *params =[URL.query componentsSeparatedByString:@"&"];
        
        NSMutableDictionary *tempDic = [NSMutableDictionary dictionary];
        for (NSString *paramStr in params) {
            NSArray *dicArray = [paramStr componentsSeparatedByString:@"="];
            if (dicArray.count > 1) {
                NSString *decodeValue = [dicArray[1] stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
                [tempDic setObject:decodeValue forKey:dicArray[0]];
            }
        }
        CGFloat r = [[tempDic objectForKey:@"r"] floatValue];
        CGFloat g = [[tempDic objectForKey:@"g"] floatValue];
        CGFloat b = [[tempDic objectForKey:@"b"] floatValue];
        CGFloat a = [[tempDic objectForKey:@"a"] floatValue];
        
        self.navigationController.navigationBar.backgroundColor = [UIColor colorWithRed:r/255.0 green:g/255.0 blue:b/255.0 alpha:a];
    }
}
```

##### WKWebView调用JS方法

`JS`执行成功还是失败会在`completionHandler`中返回。所以使用这个`API`就可以避免执行耗时的`JS`，或者`alert`导致界面卡住的问题。



```objectivec
- (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^ _Nullable)(_Nullable id, NSError * _Nullable error))completionHandler;
```

##### WKWebView中使用弹窗

如果在`WKWebView`中使用`alert`、`confirm`等弹窗，就得实现`WKUIDelegate`中相应的代理方法。



```objectivec
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {
    UIAlertController *alertCrontroller = [UIAlertController alertControllerWithTitle:@"提示" message:message preferredStyle:UIAlertControllerStyleAlert];
    [alertCrontroller addAction:[UIAlertAction actionWithTitle:@"知道了" style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
        completionHandler();
    }]];
    [self presentViewController:alertCrontroller animated:YES completion:nil];
}
```

其中`completionHandler`这个`block`一定得调用，至于在哪里调用，倒是无所谓，我们也可以写在方法实现的第一行，或者最后一行。

