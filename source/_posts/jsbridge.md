---
title: jsbridge
date: 2020-01-09 23:18:58
tags:

---

## H5 端与 安卓、ios 端的通信

基于 WebView 的机制和开放的 API, 实现这个功能有三种常见的方案：

- **API注入**，原理其实就是 Native 获取 JavaScript环境上下文，并直接在上面挂载对象或者方法，使 js 可以直接调用，Android 与 IOS 分别拥有对应的挂载方式。
- **WebView 中的 prompt/console/alert 拦截**，通常使用 prompt，因为这个方法在前端中使用频率低，比较不会出现冲突；

- **WebView URL Scheme 跳转拦截**；



### js调用 安卓 与 ios 平台

- js 调用 安卓有 3 种

  - 通过 WebView 的 `addJavascriptInterface()`进行对象映射。

  客户端
  
  ```java
  // 继承自Object类
  public class AndroidtoJs extends Object {
  
      // 定义JS需要调用的方法
      // 被JS调用的方法必须加入@JavascriptInterface注解
      @JavascriptInterface
      public void hello(String msg) {
          System.out.println("JS调用了Android的hello方法");
      }
}
  
public class MainActivity extends AppCompatActivity {
  
    WebView mWebView;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);

          mWebView = (WebView) findViewById(R.id.webview);
        WebSettings webSettings = mWebView.getSettings();
  
          // 设置与Js交互的权限
          webSettings.setJavaScriptEnabled(true);

          // 通过addJavascriptInterface()将Java对象映射到JS对象
          //参数1：Javascript对象名
          //参数2：Java对象名
          mWebView.addJavascriptInterface(new AndroidtoJs(), "test");//AndroidtoJS类对象映射到js的test对象
  
          // 加载JS代码
          // 格式规定为:file:///android_asset/文件名.html
          mWebView.loadUrl("file:///android_asset/javascript.html");
        
  ...
  ```
  
  
  
  H5 端
  
  ```html
  <!DOCTYPE html>
  <html>
     <head>
        <meta charset="utf-8">
        <title>Carson</title>  
        <script>
           
          
           function callAndroid(){
          // 由于对象映射，所以调用test对象等于调用Android映射的对象
              test.hello("js调用了android中的hello方法");
           }
        </script>
     </head>
     <body>
        //点击按钮则调用callAndroid函数
        <button type="button" id="button1" onclick="callAndroid()"></button>
     </body>
  </html>
  ```
  
  
  
  - 通过 WebViewClient 的 `shouldOverrideUrlLoading ()` 方法回调拦截 url 
  
  
  
  H5 端
  
  ```html
  <!DOCTYPE html>
  <html>
  
     <head>
        <meta charset="utf-8">
        <title>Carson_Ho</title>
        
       <script>
           function callAndroid(){
              /*约定的url协议为：js://webview?arg1=111&arg2=222*/
              document.location = "js://webview?arg1=111&arg2=222";
           }
        </script>
  </head>
  
  <!-- 点击按钮则调用callAndroid（）方法  -->
     <body>
       <button type="button" id="button1" "callAndroid()">点击调用Android代码</button>
     </body>
  </html>
  ```
  
  
  
  客户端
  
  ```java
  public class MainActivity extends AppCompatActivity {
  
      WebView mWebView;
  //    Button button;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
  
          mWebView = (WebView) findViewById(R.id.webview);
  
          WebSettings webSettings = mWebView.getSettings();
  
          // 设置与Js交互的权限
          webSettings.setJavaScriptEnabled(true);
          // 设置允许JS弹窗
          webSettings.setJavaScriptCanOpenWindowsAutomatically(true);
  
          // 步骤1：加载JS代码
          // 格式规定为:file:///android_asset/文件名.html
          mWebView.loadUrl("file:///android_asset/javascript.html");
  
  
  // 复写WebViewClient类的shouldOverrideUrlLoading方法
  mWebView.setWebViewClient(new WebViewClient() {
                                        @Override
                                        public boolean shouldOverrideUrlLoading(WebView view, String url) {
  
                                            // 步骤2：根据协议的参数，判断是否是所需要的url
                                            // 一般根据scheme（协议格式） & authority（协议名）判断（前两个参数）
                                            //假定传入进来的 url = "js://webview?arg1=111&arg2=222"（同时也是约定好的需要拦截的）
  
                                            Uri uri = Uri.parse(url);                                 
                                            // 如果url的协议 = 预先约定的 js 协议
                                            // 就解析往下解析参数
                                            if ( uri.getScheme().equals("js")) {
  
                                                // 如果 authority  = 预先约定协议里的 webview，即代表都符合约定的协议
                                                // 所以拦截url,下面JS开始调用Android需要的方法
                                                if (uri.getAuthority().equals("webview")) {
  
                                                   //  步骤3：
                                                    // 执行JS所需要调用的逻辑
                                                    System.out.println("js调用了Android的方法");
                                                    // 可以在协议上带有参数并传递到Android上
                                                    HashMap<String, String> params = new HashMap<>();
                                                    Set<String> collection = uri.getQueryParameterNames();
  
                                                }
  
                                                return true;
                                            }
                                            return super.shouldOverrideUrlLoading(view, url);
                                        }
                                    }
          );
     }
          }
  ```
  
  
  
  - 通过 WebChromeClient 的 `onJsAlert()` 、`onJsConfirm()` 、`onJsPrompt()`方法回调拦截JS对话框 `alert()` 、`confirm()`、`prompt()` 消息
  
  客户端
  
  ```java
  public class MainActivity extends AppCompatActivity {
  
      WebView mWebView;
  //    Button button;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
  
          mWebView = (WebView) findViewById(R.id.webview);
  
          WebSettings webSettings = mWebView.getSettings();
  
          // 设置与Js交互的权限
          webSettings.setJavaScriptEnabled(true);
          // 设置允许JS弹窗
          webSettings.setJavaScriptCanOpenWindowsAutomatically(true);
  
  // 先加载JS代码
          // 格式规定为:file:///android_asset/文件名.html
          mWebView.loadUrl("file:///android_asset/javascript.html");
  
  
          mWebView.setWebChromeClient(new WebChromeClient() {
                                          // 拦截输入框(原理同方式2)
                                          // 参数message:代表promt（）的内容（不是url）
                                          // 参数result:代表输入框的返回值
                                          @Override
                                          public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
                                              // 根据协议的参数，判断是否是所需要的url(原理同方式2)
                                              // 一般根据scheme（协议格式） & authority（协议名）判断（前两个参数）
                                              //假定传入进来的 url = "js://webview?arg1=111&arg2=222"（同时也是约定好的需要拦截的）
  
                                              Uri uri = Uri.parse(message);
                                              // 如果url的协议 = 预先约定的 js 协议
                                              // 就解析往下解析参数
                                              if ( uri.getScheme().equals("js")) {
  
                                                  // 如果 authority  = 预先约定协议里的 webview，即代表都符合约定的协议
                                                  // 所以拦截url,下面JS开始调用Android需要的方法
                                                  if (uri.getAuthority().equals("webview")) {
  
                                                      //
                                                      // 执行JS所需要调用的逻辑
                                                      System.out.println("js调用了Android的方法");
                                                      // 可以在协议上带有参数并传递到Android上
                                                      HashMap<String, String> params = new HashMap<>();
                                                      Set<String> collection = uri.getQueryParameterNames();
  
                                                      //参数result:代表消息框的返回值(输入值)
                                                      result.confirm("js调用了Android的方法成功啦");
                                                  }
                                                  return true;
                                              }
                                              return super.onJsPrompt(view, url, message, defaultValue, result);
                                          }
  
  // 通过alert()和confirm()拦截的原理相同，此处不作过多讲述
  
                                          // 拦截JS的警告框
                                          @Override
                                          public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
                                              return super.onJsAlert(view, url, message, result);
                                          }
  
                                          // 拦截JS的确认框
                                          @Override
                                          public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
                                              return super.onJsConfirm(view, url, message, result);
                                          }
                                      }
          );
  
  
              }
  
          }
  ```
  
  
  
  H5 端
  
  ```html
  <!DOCTYPE html>
  <html>
     <head>
        <meta charset="utf-8">
        <title>Carson_Ho</title>
        
       <script>
          
    function clickprompt(){
      // 调用prompt（）
      var result=prompt("js://demo?arg1=111&arg2=222");
      alert("demo " + result);
  }
  
        </script>
  </head>
  
  <!-- 点击按钮则调用clickprompt()  -->
     <body>
       <button type="button" id="button1" "clickprompt()">点击调用Android代码</button>
     </body>
  </html>
  ```



- js 调用 ios 平台
  - 通过设置透明的 iframe 的 src 属性

  H5端

  ```javascript
  function iOSExec() {
      ...
      if (!isInContextOfEvalJs && commandQueue.length == 1)   {
          // 如果支持 XMLHttpRequest，则使用 XMLHttpRequest 方式
          if (bridgeMode != jsToNativeModes.IFRAME_NAV) {
              ...
          } else {
              execIframe = execIframe || createExecIframe();
              execIframe.src = "gap://ready";
          }
      }
      ...
  }
  ```

  

  - 使用 XMLHttpRequest 发起请求的方式

  JS 端使用 XMLHttpRequest 发起了一个请求：`execXhr.open('HEAD', "/!gap_exec?" + (+new Date()), true);`，请求的地址是 `/!gap_exec`；并把请求的数据放在了请求的 header 里面，见这句代码：`execXhr.setRequestHeader('cmds', iOSExec.nativeFetchMessages());`。

  ```javascript
  function iOSExec() {
      ...
      if (!isInContextOfEvalJs && commandQueue.length == 1)   {
          // 如果支持 XMLHttpRequest，则使用 XMLHttpRequest 方式
          if (bridgeMode != jsToNativeModes.IFRAME_NAV) {
              // This prevents sending an XHR when there is already one being sent.
              // This should happen only in rare circumstances (refer to unit tests).
              if (execXhr && execXhr.readyState != 4) {
                  execXhr = null;
              }
              // Re-using the XHR improves exec() performance by about 10%.
              execXhr = execXhr || new XMLHttpRequest();
              // Changing this to a GET will make the XHR reach the URIProtocol on 4.2.
              // For some reason it still doesn't work though...
              // Add a timestamp to the query param to prevent caching.
              execXhr.open('HEAD', "/!gap_exec?" + (+new Date()), true);
              if (!vcHeaderValue) {
                  vcHeaderValue = /.*\((.*)\)/.exec(navigator.userAgent)[1];
              }
              execXhr.setRequestHeader('vc', vcHeaderValue);
              execXhr.setRequestHeader('rc', ++requestCount);
              if (shouldBundleCommandJson()) {
                  // 设置请求的数据
                  execXhr.setRequestHeader('cmds', iOSExec.nativeFetchMessages());
              }
              // 发起请求
              execXhr.send(null);
          } else {
            	...
          }
      }
      ...
  }
  ```





### 安卓 与 ios 平台调用js

- 安卓 调用 js 的方法有 2 种

  - 通过 `WebView的loadUrl()`

  H5 端

  ```html
  // 文本名：javascript
  <!DOCTYPE html>
  <html>
  
     <head>
        <meta charset="utf-8">
        <title>Carson_Ho</title>
        
  // JS代码
       <script>
  // Android需要调用的方法
     function callJS(){
        alert("Android调用了JS的callJS方法");
     }
  </script>
  
     </head>
  
  </html>
  ```

  

  安卓端

  ```java
  public class MainActivity extends AppCompatActivity {
  
      WebView mWebView;
      Button button;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_main);
  
          mWebView =(WebView) findViewById(R.id.webview);
  
          WebSettings webSettings = mWebView.getSettings();
  
          // 设置与Js交互的权限
          webSettings.setJavaScriptEnabled(true);
          // 设置允许JS弹窗
          webSettings.setJavaScriptCanOpenWindowsAutomatically(true);
  
          // 先载入JS代码
          // 格式规定为:file:///android_asset/文件名.html
          mWebView.loadUrl("file:///android_asset/javascript.html");
  
          button = (Button) findViewById(R.id.button);
  
  
          button.setOnClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View v) {
                  // 通过Handler发送消息
                  mWebView.post(new Runnable() {
                      @Override
                      public void run() {
  
                          // 注意调用的JS方法名要对应上
                          // 调用javascript的callJS()方法
                          mWebView.loadUrl("javascript:callJS()");
                      }
                  });
                  
              }
          });
  
          // 由于设置了弹窗检验调用结果,所以需要支持js对话框
          // webview只是载体，内容的渲染需要使用webviewChromClient类去实现
          // 通过设置WebChromeClient对象处理JavaScript的对话框
          //设置响应js 的Alert()函数
          mWebView.setWebChromeClient(new WebChromeClient() {
              @Override
              public boolean onJsAlert(WebView view, String url, String message, final JsResult result) {
                  AlertDialog.Builder b = new AlertDialog.Builder(MainActivity.this);
                  b.setTitle("Alert");
                  b.setMessage(message);
                  b.setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
                      @Override
                      public void onClick(DialogInterface dialog, int which) {
                          result.confirm();
                      }
                  });
                  b.setCancelable(false);
                  b.create().show();
                  return true;
              }
  
          });
  
  
      }
  }
  ```

  

  - 通过WebView的 `evaluateJavascript()`

  ```java
  // 只需要将第一种方法的loadUrl()换成下面该方法即可
      mWebView.evaluateJavascript（"javascript:callJS()", new ValueCallback<String>() {
          @Override
          public void onReceiveValue(String value) {
              //此处为 js 返回的结果
          }
      });
  }
  ```

  

- ios 调用 js 有一种

  - `stringByEvaluatingJavaScriptFromString`，这个方法可以让一个 UIWebView 对象执行一段 JS 代码，这样就可以达到 Objective-C 跟 JS 通信的效果

  ```objective-c
  - (void)fetchCommandsFromJs
  {
      // Grab all the queued commands from the JS side.
      NSString* queuedCommandsJSON = [_viewController.webView 
                                          stringByEvaluatingJavaScriptFromString: 
                                              @"cordova.require('cordova/exec').nativeFetchMessages()"];
  
      [self enqueCommandBatch:queuedCommandsJSON];
      if ([queuedCommandsJSON length] > 0) {
          CDV_EXEC_LOG(@"Exec: Retrieved new exec messages by request.");
      }
  }
  ```



## 总结

**为什么使用 iframe**

使用 iframe.src 发送 URL SCHEME 会有 url 长度的隐患。

创建请求，需要一定的耗时，比注入 API 的方式调用同样的功能，耗时会较长。

但是之前为什么很多方案使用这种方式呢？因为它 支持 iOS6。而现在的大环境下，iOS6 占比很小，基本上可以忽略，所以并不推荐为了 iOS6 使用这种 并不优雅 的方式。

【注】：有些方案为了规避 url 长度隐患的缺陷，在 iOS 上采用了使用 Ajax 发送同域请求的方式，并将参数放到 head 或 body 里。这样，虽然规避了 url 长度的隐患，但是 WKWebView 并不支持这样的方式。

【注2】：为什么选择 iframe.src 不选择 locaiton.href ？因为如果通过 location.href 连续调用 Native，很容易丢失一些调用。



**使用 addJavascriptInterface 的缺陷**

这个接口有漏洞，可以被不法分子利用，危害用户的安全，因此在 4.2 中引入新的接口 @JavascriptInterface（上面代码中使用的）来替代这个接口，解决安全问题。所以 Android 注入对对象的方式是 有兼容性问题的



**Cordova.js 为什么优先使用 XMLHttpRequest 的方式，以及为什么保留第二种 iframe bridge 的通信方式**

```
// XHR mode does not work on iOS 4.2, so default to IFRAME_NAV for such devices.
// XHR mode’s main advantage is working around a bug in -webkit-scroll, which
// doesn’t exist in 4.X devices anyways123
```

因为 XHR 模式在iOS 4.2上不起作用，因此此类设备的默认设置为IFRAME_NAV。



## 参考资料

https://zhuanlan.zhihu.com/p/54019800