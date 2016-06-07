---
title: iOS与JS交互
date: 2016-06-06 15:04:54
categories: "iOS"
tags: 
- UITableView
---
推荐使用WKWebView
=================
>WKWebView 是苹果在 iOS 8 中引入的新组件，目的是给出一个新的高性能的 Web View 解决方案，摆脱过去 UIWebView 的老旧笨重特别是内存占用量巨大的问题。苹果将 UIWebViewDelegate 与 UIWebView 重构成了 14 个类和 3 个协议，引入了不少新的功能和接口。

WKWebView准备工作
-------------------

<pre><code>#import <WebKit/WebKit.h>
@interface ViewController () <WKScriptMessageHandler, WKNavigationDelegate, WKUIDelegate>
@end
</pre></code>

在创建WKWebView之前,我们先做配置操作
-------------------
这边就需要用到WKWebViewConfiguration
<pre><code> WKWebViewConfiguration *config  =[[WKWebViewConfiguration alloc] init];
</pre></code>

交互需要利用WKUserContentController
-------------------

这个类是用来给JS注入对象的,对象是和网页端一起约定好的。
<pre><code>config.userContentController = [[WKUserContentController alloc] init];
</pre></code>
例如我们现在约定使用sendInfoModel这个对象,那么:
- OC中,给JS注入对象:
<pre><code>[config.userContentController addScriptMessageHandler:self name:@"sendInfoModel"];
</pre></code>
- JS中,使用对象:
这是JS的一个传值操作,通过body来传值。
<pre><code>window.webkit.messageHandlers.sendInfoModel.postMessage({body: 'JS要传值'});
</pre></code>
- 当JS通过sendInfoModel传值的时候,在iOS端,我们在下面的这个代理中接受结果:
<pre><code>-(void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message {
//首先判断一下是哪个对象(我们可以注入多个不同的对象,来进行不同的操作)

if ([message.name isEqualToString:@"sendInfoModel"]) {
// 打印所传过来的参数，只支持NSNumber, NSString, NSDate, NSArray,NSDictionary, and NSNull类

NSLog(@%@", message.body);

}
}
</pre></code>

下面利用上面的WKWebViewConfiguration *config配置构造器来创建KWWebView
-------------------

<pre><code>
self.webView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:config];

NSURL *path = [[NSBundle mainBundle] URLForResource:@"test" withExtension:@"html"];

[self.webView loadRequest:[NSURLRequest requestWithURL:path]];

[self.view addSubview:self.webView];
</pre></code>

- WKWebView的title(标题), loading(BOOL,是否在加载), estimatedProgress(加载进度),可以用KVO来监听,进行一些细节操作(进度条啥的)。


WKWebView在请求开始前会调用下面这个代理方法:
------------------

-webView:webView decidePolicyForNavigationAction:navigationAction  decisionHandler:decisionHandler
- navigationAction决定是否让一个网页被加载,我们检查它的navigationType和request这两个属性;
- navigationType 枚举值,UIWebViewNavigationTypeLinkClicked为点击链接操作;
- request用来确定它是否是外部链接。

<pre><code>- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler 

{


NSString *hostname = navigationAction.request.URL.host.lowercaseString;


if (navigationAction.navigationType == WKNavigationTypeLinkActivated
&& ![hostname containsString:@".lanou.com"])

{
// 对于跨域，需要手动跳转
[[UIApplication sharedApplication] openURL:navigationAction.request.URL];

// 不允许web内跳转

decisionHandler(WKNavigationActionPolicyCancel);

} else 

{

//允许web内跳转

self.progressView.alpha = 1.0;//(显示进度条)

decisionHandler(WKNavigationActionPolicyAllow);
}


}
</pre></code>

KWWebView完成响应
-------------------
<pre><code>-(void)webView:(WKWebView *)webView
decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse
decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler

{
//允许响应

decisionHandler(WKNavigationResponsePolicyAllow);

//若为不允许响应,那么web内容就传不过来


}
</pre></code>

针对HTTPS协议的链接,加载时都会触发以下方法来验证证书(操作与使用AFN进行HTTPS验证证书一样)
-------------------

<pre><code>-(void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:
(void (^)(NSURLSessionAuthChallengeDisposition disposition,
NSURLCredential *__nullable credential))completionHandler 

{


//如不需要验证传默认的就可以

completionHandler(NSURLSessionAuthChallengePerformDefaultHandling, nil);
}
</pre></code>

WKUIDelegate
-------------------

与JS原生的alert,confirm,prompt进行交互:
- alert :
JS中:
<pre><code>function callJsAlert() {
alert('这个是OC调用JS的方法,并且通过Alert()进行显示出来!');

window.webkit.messageHandlers.sendInfoModel.postMessage({body: '在JS中调用JS alert中方法'});
}
</pre></code>
OC中: 当JS调用alert时会触发此方法
<pre><code>- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message
initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler 
{
UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"alert" message:@"JS调用alert" preferredStyle:UIAlertControllerStyleAlert];
[alert addAction:[UIAlertAction actionWithTitle:@"确定" style: UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
completionHandler();
}]];

[self presentViewController:alert animated:YES completion:NULL];
NSLog(@"%@", message);
}
</pre></code>

- confirm
- prompt

与alert相似

JS中:
<pre><code>
function callJsConfirm() {
if (confirm('confirm', 'Objective-C call js to show confirm')) {
document.getElementById('jsParamFuncSpan').innerHTML
= 'true';
// sendInfoModel是我们所注入的对象
window.webkit.messageHandlers.sendInfoModel.postMessage({body: '我是JS里面的内容'});//传值
} else {
document.getElementById('jsParamFuncSpan').innerHTML
= 'false';
}

}

function callJsInput() {
var response = prompt('Hello', '请输入你的名字:');
document.getElementById('jsParamFuncSpan').innerHTML = response;

// sendInfoModel是我们所注入的对象
window.webkit.messageHandlers.sendInfoModel.postMessage({body: response});
}
</pre></code>

OC中:

<pre><code>- (void)webView:(WKWebView *)webView
runJavaScriptConfirmPanelWithMessage:(NSString *)message
initiatedByFrame:(WKFrameInfo *)frame
completionHandler:(void (^)(BOOL result))completionHandler

{

UIAlertController *alert = [UIAlertController alertControllerWithTitle:
@"confirm" message:@"JS调用confirm"
preferredStyle:UIAlertControllerStyleAlert];
[alert addAction:[UIAlertAction actionWithTitle:@"确定"
style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action){
completionHandler(YES);
}]];
[alert addAction:[UIAlertAction actionWithTitle:@"取消"
style:UIAlertActionStyleCancel handler:^(UIAlertAction * _Nonnull action) {
completionHandler(NO);
}]];
[self presentViewController:alert animated:YES completion:NULL];
NSLog(@"%@", message);

}

-(void)webView:(WKWebView *)webView
runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt
defaultText:(nullable NSString *)defaultText
initiatedByFrame:(WKFrameInfo *)frame
completionHandler:(void (^)(NSString * __nullable result))completionHandler

{

NSLog(@"%@", prompt);
UIAlertController *alert = [UIAlertController alertControllerWithTitle:
@"textinput" message:@"JS调用输入框"
preferredStyle:UIAlertControllerStyleAlert];
[alert addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
textField.textColor = [UIColor redColor];
}];
[alert addAction:[UIAlertAction actionWithTitle:@"确定"
style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
completionHandler([[alert.textFields lastObject] text]);
}]];

[self presentViewController:alert animated:YES completion:NULL];

}
</pre></code>