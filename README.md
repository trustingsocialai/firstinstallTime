一、基本的使用方法
介绍：
WKWebView是苹果推出的框架，性能比UIWebView更优。 首先上DEMO
WeiCountInfoParSAtest-phl-api.fyinformation.ccWeiCountInfoParEN
WKWebView有两个delegate,WKUIDelegate 和 WKNavigationDelegate。
WKNavigationDelegate主要处理一些跳转、加载处理操作，
WKUIDelegate主要处理JS脚本，确认框，警告框等。因此WKNavigationDelegate更加常用。

    // 1.配置环境
    WKWebViewConfiguration * configuration = [[WKWebViewConfiguration alloc]init];
    userContentController =[[WKUserContentController alloc]init];
    configuration.userContentController = userContentController;
    wkWebView = [[WKWebView alloc]initWithFrame:CGRectMake(0, 0, [UIScreen mainScreen].bounds.size.width, [UIScreen mainScreen].bounds.size.height) configuration:configuration];
    
    // 2.注册方法（必要设置，不然WKWebView会无法释放）
    WKDelegateController * delegateController = [[WKDelegateController alloc]init];
    delegateController.delegate = self;
    // 有添加一定有移除，成对出现
    [userContentController addScriptMessageHandler:delegateController  name:@"NativeMethod"];
    
    [self.view addSubview:wkWebView];
    
    wkWebView.UIDelegate = self;
    wkWebView.navigationDelegate = self;
    
    NSURL *url = [NSURL URLWithString:@"http://www.baidu.com"];
    // 根据URL创建请求
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    [request setTimeoutInterval:15];
    [wkWebView loadRequest:request];
常用的代理方法

WKNavigationDelegate 方法较为常用

   // 页面开始加载时调用 
    -(void)webView:(WKWebView *)webView didStartProvisionalNavigation:(WKNavigation *)navigation{
    }

    // 当内容开始返回时调用
    -(void)webView:(WKWebView *)webView didCommitNavigation:(WKNavigation *)navigation{
    }

   // 页面加载完成之后调用
    -(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation{
    }

    // 页面加载失败时调用
    -(void)webView:(WKWebView *)webView didFailProvisionalNavigation:(WKNavigation *)navigation{
    }

    // 接收到服务器跳转请求之后调用
    -(void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(WKNavigation *)navigation{
    }

    // 在收到响应后，决定是否跳转
    -(void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler{

        NSLog(@"%@",navigationResponse.response.URL.absoluteString);
        //允许跳转
        decisionHandler(WKNavigationResponsePolicyAllow);
        //不允许跳转
        //decisionHandler(WKNavigationResponsePolicyCancel);
    }

    // 在发送请求之前，决定是否跳转
    -(void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler{

        NSLog(@"%@",navigationAction.request.URL.absoluteString);
        //允许跳转
        decisionHandler(WKNavigationActionPolicyAllow);
        //不允许跳转
        //decisionHandler(WKNavigationActionPolicyCancel);
    }
WKUIDelegate

    // 创建一个新的WebView
    -(WKWebView *)webView:(WKWebView *)webView createWebViewWithConfiguration:(WKWebViewConfiguration *)configuration forNavigationAction:(WKNavigationAction *)navigationAction windowFeatures:(WKWindowFeatures *)windowFeatures{

        return [[WKWebView alloc]init];
    }

    // 输入框
    -(void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(nullable NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * __nullable result))completionHandler{

        completionHandler(@"http");
    }

    // 确认框
    -(void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL result))completionHandler{

        completionHandler(YES);
    }

    // 警告框
    -(void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler{

        NSLog(@"%@",message);
        completionHandler();
    }

    -(void)dealloc{

        //这里需要注意，前面增加过的方法一定要remove掉。
        [userContentController removeScriptMessageHandlerForName:@"NativeMethod"];
    }
WKScriptMessageHandler

    -(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{

        NSLog(@"name:%@\\\\n body:%@\\\\n frameInfo:%@\\\\n",message.name,message.body,message.frameInfo);
    }


// 创建新的控制器设置代理（解决不能释放的问题）
    @protocol WKDelegate <NSObject><br>
    -(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message;
    @end

    @interface WKDelegateController : UIViewController <WKScriptMessageHandler>
    @property (weak , nonatomic) id<WKDelegate> delegate;
    @end

    .m文件中的实现

    -(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{

        if ([self.delegate respondsToSelector:@selector(userContentController:didReceiveScriptMessage:)]) {
            [self.delegate userContentController:userContentController didReceiveScriptMessage:message];
        }
    }
关于session 同步 cookies的问题

#### 1.基本配置
    NSMutableString *cookies = [NSMutableString string];
    WKUserScript * cookieScript = [[WKUserScript alloc] initWithSource:[cookies copy]
                                                         injectionTime:WKUserScriptInjectionTimeAtDocumentStart
                                                      forMainFrameOnly:NO];
    [userContentController addUserScript:cookieScript];
    
    // 一下两个属性是允许H5视屏自动播放,并且全屏,可忽略
    configuration.allowsInlineMediaPlayback = YES;
    configuration.mediaPlaybackRequiresUserAction = NO;
    // 全局使用同一个processPool
    configuration.processPool = [[WKWebKitSupport sharedSupport] processPool];
    configuration.userContentController = userContentController;

#### 2.保存到本地
    // 在收到响应后，决定是否跳转<br>
    -(void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler{

        NSLog(@"%@",navigationResponse.response.URL.absoluteString);
        NSHTTPURLResponse *response = (NSHTTPURLResponse *)navigationResponse.response;
        // 获取cookie,并设置到本地
        NSArray *cookies =[NSHTTPCookie cookiesWithResponseHeaderFields:[response allHeaderFields] forURL:response.URL];
        for (NSHTTPCookie *cookie in cookies) {
            [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookie:cookie];
        }

        //允许跳转
        decisionHandler(WKNavigationResponsePolicyAllow);
        //不允许跳转
        //decisionHandler(WKNavigationResponsePolicyCancel);
    }

 #### 3.在开始请求时注入

    NSURL *url = [NSURL URLWithString:urlString];
    NSMutableString *cookies = [NSMutableString string];
    NSMutableURLRequest *requestObj = [NSMutableURLRequest requestWithURL:url cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:10];

    // 一般都只需要同步JSESSIONID,可视不同需求自己做更改
    NSString * JSESSIONID;
    // 获取本地所有的Cookie
    NSArray *tmp = [[NSHTTPCookieStorage sharedHTTPCookieStorage] cookies];
        for (NSHTTPCookie * cookie in tmp) {
            if ([cookie.name isEqualToString:@"JSESSIONID"]) {
                JSESSIONID = cookie.value;
                break;
            }
        }
     if (JSESSIONID.length) {
          // 格式化Cookie
          [cookies appendFormat:@"JSESSIONID=%@;",JSESSIONID];
      }
    // 注入Cookie
    [requestObj setValue:cookies forHTTPHeaderField:@"Cookie"];
    // 加载请求
    [self.wk_webView loadRequest:requestObj];
新增拨打电话和弹窗

    // 在发送请求之前，决定是否跳转<br>
    -(void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler{

        NSLog(@"%@",navigationAction.request.URL.absoluteString);
        // 拨打电话

        NSURL *URL = navigationAction.request.URL;
        NSString *scheme = [URL scheme];
        UIApplication *app = [UIApplication sharedApplication];
        // 打电话
        if ([scheme isEqualToString:@"tel"]) {
            if ([app canOpenURL:URL]) {
                [app openURL:URL];
                // 一定要加上这句,否则会打开新页面
                decisionHandler(WKNavigationActionPolicyCancel);
                return;
            }
        }
        // 打开appstore
        if ([URL.absoluteString containsString:@"ituns.apple.com"]) {
            if ([app canOpenURL:URL]) {
                [app openURL:URL];
                decisionHandler(WKNavigationActionPolicyCancel);
                return;
            }
        }

        //允许跳转
        decisionHandler(WKNavigationActionPolicyAllow);
        //不允许跳转
        //decisionHandler(WKNavigationActionPolicyCancel);
    }

    #### 警告框
    -(void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler{

        NSLog(@"%@",message);
        //  js 里面的alert实现，如果不实现，网页的alert函数无效  ,
        UIAlertController *alertController = [UIAlertController alertControllerWithTitle:message message:nil preferredStyle:UIAlertControllerStyleAlert];
        [alertController addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction *action) {
            completionHandler();
        }]];
        [alertController addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction *action){
            completionHandler();
        }]];

        [self presentViewController:alertController animated:YES completion:^{}];
        // 要实现
    //    completionHandler();
    }

    #### 确认框

    -(void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL))completionHandler {

        //  js 里面的alert实现，如果不实现，网页的alert函数无效  ,
        UIAlertController *alertController = [UIAlertController alertControllerWithTitle:message message:nil preferredStyle:UIAlertControllerStyleAlert];
        [alertController addAction:[UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleDefault handler:^(UIAlertAction *action) {
            completionHandler(YES);
        }]];
        [alertController addAction:[UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:^(UIAlertAction *action){
            completionHandler(NO);
        }]];

        [self presentViewController:alertController animated:YES completion:^{}];

    }
适配HTML5页面的问题

    // 内容视图自适应大小
    NSString *jScript = @"var meta = document.createElement('meta'); meta.setAttribute('name', 'viewport'); meta.setAttribute('content', 'width=device-width'); document.getElementsByTagName('head')[0].appendChild(meta);";
    WKUserScript *wkUScript = [[WKUserScript alloc] initWithSource:jScript injectionTime:WKUserScriptInjectionTimeAtDocumentEnd forMainFrameOnly:YES];
    [userContentController addUserScript:wkUScript];
