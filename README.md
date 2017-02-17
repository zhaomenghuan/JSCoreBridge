<!--
# iPhuan Open Source
# JSCoreBridge
# Created by iPhuan on 2017/2/16.
# Copyright © 2017年 iPhuan. All rights reserved.
-->

JSCoreBridge
=============================================================
JSCoreBridge是基于iOS平台[Apache Cordova](http://cordova.apache.org/)修改的开源框架，Cordova的用处在于作为桥梁通过插件的方式实现了Web与Native之间的通信，而JSCoreBridge参考其进行删减修改（移除了开发者在平时用不上的类和方法），改写了其传统的通信机制，在保留了Cordova实用的功能前提下，精简优化了框架占用大小，并且省去了繁琐的工程设置选项，通过的新的实现方式大大提供了通信效率。JSCoreBridge开源框架力在为开发者提供更便捷的Hybird开发体验。


用途
-------------------------------------------------------------
适用于Hybird开发者，希望通过JSCoreBridge框架实现客户端Web与Native之间的交互与通信。


通信原理
-------------------------------------------------------------
### Cordova通信原理：

1. Web创建自定义scheme “`gap://ready`”，并响应链接跳转事件；
2. Cordova通过WebView代理方法`webView:shouldStartLoadWithRequest:navigationType`截获该gap跳转
3. Cordova通过WebView `stringByEvaluatingJavaScriptFromString`方法执行Cordova JS方法`nativeFetchMessages`获取Web当前的命令参数并转化为`CDVInvokedUrlCommand`对象；
4. Cordova根据`CDVInvokedUrlCommand`对象的`className`和`methodName`属性找到对应插件和对应的插件方法，并执行插件方法；
5. Cordova执行完插件方法后如需给Web返回数据结果，则再次通过WebView `stringByEvaluatingJavaScriptFromString`方法执行Cordova JS方法`nativeCallback`，通过`CDVInvokedUrlCommand`的`callbackId`作为标识将结果发送给Web对应的回调。

### JSCoreBridge通信原理：

不在使用传统的scheme链接跳转截取和`stringByEvaluatingJavaScriptFromString`执行JS的方法，通过iOS7新增的**`JavaScriptCore.framework`**来实现JS和Native之间的通信。

1. Web调用`jsCoreBridge.js`的`exec`或者`execSync`方法直接将命令参数传给客户端；
2. JSCoreBridge将命令参数转化为`JSCInvokedPluginCommand`对象；
3. JSCoreBridge根据`JSCInvokedPluginCommand`对象的`className`和`methodName`属性找到对应插件和对应的插件方法，并执行插件方法；
4. JSCoreBridge执行完插件方法后如需给Web返回数据结果，直接调用`jsCoreBridge.js`的`nativeCallback`方法，通过`JSCInvokedPluginCommand`的`callbackId`作为标识将结果发送给Web对应的回调。


如何获取JSCoreBridge
-------------------------------------------------------------
1. 直接在GitHub上[获取](https://github.com/iPhuan/JSCoreBridge.git)
2. 通过CocoaPods添加到工程：  

> * 如果你想使用完整版的JSCoreBridge，添加以下命令行到Podfile：  

```ruby
    pod 'JSCoreBridge'
```

> * 如果你想使用Lite版的JSCoreBridge，添加以下命令行到Podfile：  

```ruby
    pod 'JSCoreBridge/JSCoreBridgeLite'
```

注：Lite版的JSCoreBridge将不使用`config.xml`进行功能选项配置，JSCoreBridgeLite仅仅实现了最基本的通信。
  


使用说明
=============================================================
JSCoreBridge框架可通过CocoaPods Pod到工程，也可手动下载源码添加，加入JSCoreBridge后，简单配置config.xml和jsCoreBridge.js即可使用，如为手动添加，需添加`JavaScriptCore.framework`库。config.xml和jsCoreBridge.js的相关说明下文会做详细介绍。

[JSCoreBridge Demo](https://github.com/iPhuan/JSCoreBridge.git)中有JSCoreBridge的详细使用样例代码，可下载参考使用。

JSCoreBridge Web平台
-------------------------------------------------------------
### jsCoreBridge.js存放说明：  

* jsCoreBridge.js本身在工程当中，如打开的html文件在bundle中，可直接引用，当然如果你的html文件在bundle的子目录下，你希望jsCoreBridge.js和你的网页目录在同一级，你也可以将jsCoreBridge.js拷贝到该同级目录；
* 如果你的html文件存储在沙盒，请务必把jsCoreBridge.js拷贝到沙盒；
* 如果你的网页在远程网站上，那么你同样需要将jsCoreBridge.js放到你的远程网站上；
jsCoreBridge.js的使用原则在于，保证你的html文件能够引用到。

<br />
### jsCoreBridge.js接口说明：

jsCoreBridge.js对应于Cordova的[cordova.js](https://github.com/apache/cordova-ios/blob/master/CordovaLib/cordova.js)通过`jsCoreBridge`对象来调用，也兼容Cordova用法，可以通过`cordova`对象调用，jsCoreBridge接口如下：  

* **`jsCoreBridge.version`**  

> 获取当前JSCoreBridge Web平台JS版本号。<br />
> 客户端JSCoreBridge框架对jsCoreBridge.js有最低版本要求，Pod到工程的jsCoreBridge.js相对于当前客户端jsCoreBridge框架都是最新的版本，可放心使用，如果你自行从其他途径下载jsCoreBridge.js，请保证该版本能够兼容客户端jsCoreBridge框架。

* **`jsCoreBridge.exec`**  

> 执行客户端对应插件方法。<br />
> 通过该方法可以告诉客户端JSCoreBridge框架通过对应插件的对应方法去执行相应的事情，代码示例如下：

```javascript
    var params = {title: 'JSCoreBridge Demo'};

    jsCoreBridge.exec(function (res) {
    // 成功回调
    }, function (err) {
    // 失败回调
    }, 'JSCTestPlugin', 'changeNavTitle', [params]);
```

   > - 第一个函数为成功回调，第二个函数为失败回调，通过res和err获取结果数据，当然如果你不想收到回调，这两个参数可以传空，如果你不希望接收res和err结果数据，回调函数你也可以不用带参数；
   > - `JSCTestPlugin`为客户端对应的插件Plugin类名；
   > - `changeNavTitle`为JSCTestPlugin插件中对应的插件方法；
   > - 最后一个参数则为Web传给客户端的参数，通过数组的方式传递，至于数组里面传递什么样的数据，由开发者自行决定，当然该参数你也可以传空或者不传。

* **`jsCoreBridge.execSync`**  

> 同步执行客户端对应插件方法。<br />
> 与exec接口不同的是该方法为同步操作，所有没有成功与失败回调函数，其代码示例如下：  

```javascript
    var version = jsCoreBridge.execSync('JSCTestPlugin', 'getAppVersionSync', null);
```

* **`deviceready`**  

> JSCoreBridge运行环境已准备好监听事件。<br />
> 可通过以下示例代码来监听JSCoreBridge准备完成：  

```javascript
    document.addEventListener('deviceready', onDeviceReady, false)
```
:warning: 注意：为了保证客户端插件方法能够正确执行，请在`deviceready`执行后调用jsCoreBridge对象的方法；如果你在`deviceready`回调中调用`jsCoreBridge.exec`，不要企望客户端对应插件方法会在`jsCoreBridgeDidReady:`之前调用，`jsCoreBridge.exec`为异步操作，除非你使用`jsCoreBridge.execSync`方法。

<br />

* **`pause`**  

> 客户端已经进入后台监听事件。<br />
> 可通过以下示例代码来监听客户端已经进入后台：

```javascript
    document.addEventListener('pause', onPause, false)
```

* **`resume`**  

> 客户端即将进入前台监听事件。<br />
> 可通过以下示例代码来监听客户端即将进入前台：

```js
    document.addEventListener('resume', onResume, false)
```



JSCoreBridge Native平台
-------------------------------------------------------------
### [config.xml：](http://cordova.apache.org/docs/en/latest/config_ref/index.html)  

在Cordova中config.xml是框架功能选项的配置文件，包含工程的一些信息，插件白名单，Web页面访问白名单，WebView属性设置等。同样在JSCoreBridge中，我们将config.xml移植了过来，并对一些配置选项进行了删减，以便达到一个轻量级的JSCoreBridge框架。  

config.xml文件并不是必须的，当你使用`JSCoreBridgeLite`时，将不在使用config.xml文件来配置框架；当然你也可以通过设置[JSCWebViewController](#JSCWebViewController)类的`configEnabled`属性来关闭使用config.xml，以使用一个最轻量化的JSCoreBridge。  

想了解config.xml文件如何配置，可进一步点击[这里](http://cordova.apache.org/docs/en/latest/config_ref/index.html)，到Cordova官方网站进行了解。  
当然对于一般的开发者来说，JSCoreBridge当中的config.xml样例已足够满足需求，你只需配置插件白名单即可，配置示例如下：  

```xml
    <feature name="JSCTestBasePlugin">
    <param name="ios-package" value="JSCTestBasePlugin" />
    <param name="onload" value="true" />
    </feature>
```  
> 一般来说保持`feature`当中`name`的值和`param`当中`value`值一致，当然你也可以不一致，但必须保证`param`当中`value`值和对应的插件类名一致；
> 如果希望插件在JSCoreBridge初始化时就加载，可以通过`<param name="onload" value="true" />`来设置，如果不需要，可以省去该行。  


在JSCoreBridge中，以下配置选项目前暂未实现：  

1. [content](http://cordova.apache.org/docs/en/latest/config_ref/index.html#content)
2. [access](http://cordova.apache.org/docs/en/latest/config_ref/index.html#access)
3. [engine](http://cordova.apache.org/docs/en/latest/config_ref/index.html#engine)
4. [plugin](http://cordova.apache.org/docs/en/latest/config_ref/index.html#plugin)
5. [variable](http://cordova.apache.org/docs/en/latest/config_ref/index.html#variable)
6. [preference](http://cordova.apache.org/docs/en/latest/config_ref/index.html#preference)中 (`BackupWebStorage`， `TopActivityIndicator`，  `ErrorUrl`， `OverrideUserAgent`， `AppendUserAgent`， `target-device`， `deployment-target`， `CordovaWebViewEngine`， `SuppressesLongPressGesture`， `Suppresses3DTouchGesture`)

在JSCoreBridge中，以下配置选项不再需要添加：  

1. [widget](http://cordova.apache.org/docs/en/latest/config_ref/index.html#widget)中 (`id`， `version`，`defaultlocale`，`ios-CFBundleVersion`，`xmlns`，`xmlns:cdv`)
2. [name](http://cordova.apache.org/docs/en/latest/config_ref/index.html#name)
3. [description](http://cordova.apache.org/docs/en/latest/config_ref/index.html#description)
4. [author](http://cordova.apache.org/docs/en/latest/config_ref/index.html#author)  

:warning: 如工程用到`config.xml`，请在`JSCoreBridge/optional`目录下将`config.xml`复制到其他目录并添加到工程使用；


<br />
### <a name="JSCWebViewController">JSCWebViewController：</a> 
JSCWebViewController是JSCoreBridge框架直接供开发者使用的ViewController，可以直接使用，也可根据自己的需求来继承使用，其部分API说明如下：  

* **`bridgeDelegate`**   

> JSCoreBridge代理。可通过该对象执行相应代理方法，具体可参考[JSCBridgeDelegate](#JSCBridgeDelegate)。  


* **`configFilePath`**   

> config.xml文件路径。默认从Bundle根目录获取，如果设置该属性，则从该路径获取，不支持网络地址。


* **`configEnabled`**   

> 是否开启config配置功能。默认开启，如需关闭，可设置为NO；当使用JSCoreBridgeLite时`configEnabled`属性不可设置，始终为关闭状态。


* **`shouldAutoLoadURL`**   

> 是否自动加载URL。默认自动加载通过`initWithUrl:`初始化的URL，设置为NO关闭自动加载。  


```objective-c
* **`- (instancetype)initWithUrl:(NSString *)url`**  
```

> 通过字符串链接初始化URL。可在`JSCWebViewController`子类中重写该方法。  


* **`- (void)loadURL:(NSURL *)URL`**  
* **`- (void)loadHTMLString:(NSString *)htmlString`**   

> 通过调用以上两方法进行网页手动加载  


* **`- (void)jsCoreBridgeWillReady:(UIWebView *)webView`**  
* **`- (void)jsCoreBridgeDidReady:(UIWebView *)webView`**  

> JSCoreBridge将要准备就绪和已准备就绪回调。分别在`deviceready`通知发送之前和之后调用，方便开发者在这两个时刻进行相应操作，可在[JSCWebViewController](#JSCWebViewController)子类中重写该方法使用。  


:warning: **特别提示：**关于客户端Native及Web的相应回调方法的执行顺序请参考[网页加载回调执行顺序说明](#WebLoadOrder)。  


<br />
### <a name="JSCBridgeDelegate">JSCBridgeDelegate：</a>
JSCBridgeDelegate是JSCoreBridge的代理，可通过该代理向Web发送结果数据，执行JS等方法。该代理作为[JSCWebViewController](#JSCWebViewController)和[JSCPlugin](#JSCPlugin)的属性来使用。

* **`- (void)registerPlugin:(JSCPlugin *)plugin withPluginName:(NSString *)pluginName`**  

> 将已有的插件通过类名注册到插件白名单当中。如果使用`config.xml`，那么JSCoreBridge将只会识别`config.xml`配置好的插件白名单，不在白名单范围内的的插件将不予加载使用，可通过该方法将插件注册到白名单当中去。  


* **`- (nullable __kindof JSCPlugin *)getPluginInstance:(NSString *)pluginName`**  

> 通过插件类名来获取插件对象。可通过该方法获取对应Plugin的实例对象。  


* **`- (void)sendPluginResult:(JSCPluginResult *)result forCallbackId:(NSString *)callbackId`**  

> 向Web发送结果数据。将结果数据以[JSCPluginResult](#JSCPluginResult)对象实例进行封装，并以`callbackId`作为回调标识发送给Web。代码实例如下：  

```objective-c
    NSDictionary *message = @{@"resCode":@"0", @"resMsg":@"OK"};
    // 将要返回给Web的结果以字典形式通过JSCPluginResult初始化
    JSCPluginResult *result = [JSCPluginResult resultWithStatus:JSCCommandStatus_OK messageAsDictionary:message];
    // 发送结果
    [self.bridgeDelegate sendPluginResult:result forCallbackId:command.callbackId];
```  


* **`- (JSValue *)evaluateScript:(NSString *)script`**  
* **`- (JSValue *)callScriptFunction:(NSString *)funcName withArguments:(nullable NSArray *)arguments`**  

> 执行JS和调用JS函数方法。方法一实际通过`JSContext`的`evaluateScript:`方法执行JS；方法二通过`JSContext`的`callWithArguments:`调用JS函数，需要传递函数名称`funcName`，如果该函数直属于Window对象，则直接传递函数名，如果该函数并不直属于Window对象，则可通过键值路径的方式调用，代码示例如下： 

```objective-c
    [self.bridgeDelegate callScriptFunction:@"jsCoreBridge.fireDocumentEvent" withArguments:@[@"deviceready"]];
```  
  
> 其中`jsCoreBridge`必须为Window的属性。


* **`- (void)onMainThreadEvaluateScript:(NSString *)script`**  
* **`- (void)onMainThreadCallScriptFunction:(NSString *)funcName withArguments:(nullable NSArray *)arguments`**  

> 与上两方法作用一致，只是这两个方法确保执行JS和调用JS函数在主线程上。在特定的情况下不在主线程上执行JS将导致程序崩溃，通过这两个方法可以解决该问题。  


* **`- (void)runInBackground:(void (^)())block`**  
* **`- (void)runOnMainThread:(void (^)())block`**  

> 辅助类方法，方便开发者在后台或者主线程上处理对应事情。 



<br />
### <a name="JSCPlugin">JSCPlugin：</a>
JSCPlugin即为我们刚刚一直说的插件，这是一个基类，开发者需根据需求来分类建立多个插件，而这些插件都应当要继承于JSCPlugin来使用，才能保障插件的正常执行。  
JSCPlugin插件方法的声明示例如下：  

```objective-c
    - (void)changeNavTitle:(JSCInvokedPluginCommand *)command;
    - (void)sendEmail:(JSCInvokedPluginCommand *)command;
    - (NSString *)getAppVersionSync:(JSCInvokedPluginCommand *)command;
```   

> 接收一个[JSCInvokedPluginCommand](#JSCInvokedPluginCommand)对象，JSCoreBridge支持同步操作，如果需要使用同步操作，需要对应有一个返回值，而该返回值必须是一个Object对象，否则将给Web返回空的结果数据。


JSCPlugin的部分API说明如下：  

* **`webView`**   
* **`webViewController`**   

> 获取当前`webView`和`webViewController`。  


* **`backupCallbackId`**   

> 用于备份某个插件方法的`callbackId`。当你在某个插件方法中使用了某个对象，而该对象的一些操作需要在代理方法中获取结果时，可通过该属性来保存当前插件方法的回调`callbackId`，以便在代理回调中继续使用该`callbackId`来发送结果数据。具体用法可参看[JSCoreBridge Demo](https://github.com/iPhuan/JSCoreBridge.git)。  
> 当然该用法只适用于当前插件只有一个插件方法需要用到backupCallbackId，如果多个插件方法需要保存callbackId，建议参考cordova官方插件的一些写法，将`callbackId`作为其对应使用对象的属性成员，如[CDVCamera](https://github.com/apache/cordova-plugin-camera/blob/master/src/ios/CDVCamera.h)插件，`CDVCameraPicker`继承`UIImagePickerController`，并拥有`callbackId`属性。  


* **`- (void)pluginDidInitialize`**  

> 插件初始化后回调该方法，可在该方法中进行一些初始化的操作，类似于`UIViewController`的`viewDidLoad`方法。  


* **`- (void)canCallPlugin`**  

> JSCoreBridge调用插件方法时，先通过该方法进行验证，如果返回YES，则可正常调用插件，如返回NO，则无法调用。开发者可通过该方法进行一些权限的条件设置。  


<br />
### <a name="JSCPluginResult">JSCPluginResult：</a>
JSCoreBridge给Web发送的结果数据通过JSCPluginResult对象进行封装，可以封装成字符串，数组，Cordova特定的格式等多种数据格式进行发送。  


* **`status`**  

> 结果状态，`JSCCommandStatus_OK`时将结果发送给成功回调，`JSCCommandStatus_ERROR`时将结果发送给失败回调。  


* **`keepCallback`**  

> 是否需要继续回调。默认为NO，同一个`callbackId`只能发送一次结果数据，设为YES，则支持多次回调。比如写一个监听客户端某个按钮的点击事件的插件方法，用户点击按钮一次，给Web发送一次结果消息，此种场景就需要将`keepCallback`设置为YES。 



<br />
### <a name="JSCInvokedPluginCommand">JSCInvokedPluginCommand：</a>
JSCoreBridge通过JSCInvokedPluginCommand对象将Web发送给Native的命令参数进行封装，其属性包含如下成员：   


* **`callbackId`** 
* **`className`**  
* **`methodName`**  
* **`arguments`**  

分别为回调的callbackId标识，插件类名，插件方法名，Web传给客户端的参数，JSCoreBridge正是通过这些属性来完成Web交给Native的任务。  


<br />
### 其他框架类：  
对于框架其他的类，默认为私有状态，建议开发者不要随意调用，或者随意修改，在使用框架的过程中如遇任何问题和bug欢迎[联系本人](#ContactInfo)沟通商讨解决。  



<a name="WebLoadOrder">网页加载回调执行顺序说明：</a>
-------------------------------------------------------------
关于JSCoreBridge加载网页时，Web和Native对应回调方法的执行顺序，这里需要特别说明下：  

* **如果`jsCoreBridge.js`在html页面直接引用，如下所示：**  

```js
    <script type="text/javascript" src="jscorebridge.js"></script>
```  
各个回调的执行顺序如下：  

1. `jsCoreBridgeWebView:didCreateJavaScriptContext` 
> `JSCBridge`类中JSContext创建后的回调方法。  

2. `window.onload`  
> 如果你在Web写了该方法。  

3. `jscWindowOnLoad`  
> `JSCBridge`类中向Web添加的Web load的通知回调，`window.addEventListener("load", jscWindowOnLoad, false)`。  

4. `jsCoreBridgeWebViewDidFinishLoad:`
> [JSCWebViewController](#JSCWebViewController)类中回调方法，实为`WebView`的`webViewDidFinishLoad:`代理方法。  

5. `jsCoreBridgeWillReady:`
> JSCoreBridge即将准备就绪时的回调  

6. `deviceready`
> Native向Web发送的JSCoreBridge已准备就绪的通知事件，Web通过该事件名来监听

7. `jsCoreBridgeDidReady:`
> JSCoreBridge准备就绪之后的回调   


<br />
* ** 如果`jsCoreBridge.js`是在别的JS通过appendChild的方式加入，如下所示：**   

```js
    var script = document.createElement('script');
    script.type = 'text/javascript';
    script.src = 'jscorebridge/jscorebridge.js';
    var head = document.getElementsByTagName('head')[0];
    head.appendChild(script);
```   

各个回调的执行顺序如下：  

1. `jsCoreBridgeWebView:didCreateJavaScriptContext` 
2. `jsCoreBridgeWebViewDidFinishLoad:`
3. `window.onload`  
4. `jscWindowOnLoad`  
5. `jsCoreBridgeWillReady:`
6. `deviceready`
7. `jsCoreBridgeDidReady:`  

> 与第一种情况不同的是，当`jsCoreBridge.js`通过代码的方式加入后，`WebView`并不会等待`jsCoreBridge.js`加载完后再回调`webViewDidFinishLoad:`。JSCoreBridge在实现上没有选择在`jsCoreBridgeWebView:didCreateJavaScriptContext`时就发送`deviceready`通知而是选择了在`jscWindowOnLoad`中发送的原因有二：一是考虑到第二种执行情况，在`jsCoreBridgeWebView:didCreateJavaScriptContext`中`jsCoreBridge.js`并没有加载完毕，此时无法使用`jsCoreBridge`对象；二是，如果在`window.onload`中发送`deviceready`通知，那么对于Web开发人员来说他如果再写了`window.onload`方法，该方法将不再调用。  


开发者可参考以上两种情况的执行顺序来决定自己在开发中如何在各个回调中处理相应事情。



:warning: 风险声明
-------------------------------------------------------------
* JSCoreBridge框架使用开源类[UIWebView+TS_JavaScriptContext](https://github.com/TomSwift/UIWebView-TS_JavaScriptContext)，JSCoreBridge修改后的类为`UIWebView+JSCJavaScriptContext`，该类中的`- (void)webView:(id)unused didCreateJavaScriptContext:(JSContext *)ctx forFrame:(id<JSCWebFrame>)frame`回调方法，使用了`parentFrame`协议方法，该方法可能会被认为是私有API而导致您的APP被苹果拒绝，如果您对该问题有所介意，请勿使本框架。当然JSCoreBridge会一直跟进和更新，之后有更好的实现方法，会第一时间解决该风险。  

* 本框架虽然已进行各多次自测，但是并未进行大范围的试用，避免不了会有未知的bug产生，如果您使用本框架，那么该风险您需要自行承担。同事也欢迎您给本人反馈在使用中遇到的问题和bug。  


开源说明
=============================================================
本框架是本人在深入了解[Apache Cordova](http://cordova.apache.org/)后在此基础上修改封装的，本着开源的思想，现上传至[GitHub](https://github.com/iPhuan/JSCoreBridge.git)，并提供CocoaPods支持，之后会一直跟进更新，如果您在使用本框架，欢迎及时反馈您在使用过程中遇到的各种问题和bug，也欢迎大家跟本人沟通和分享更多互联网技术。更多开源资源将会不定期的更新至[iPhuanOpenSource](https://github.com/iPhuan/iPhuanOpenSource.git)  


<a name="ContactInfo">如何联系我</a>
=============================================================
邮箱：iphuan@qq.com  
QQ：519310392 （添加QQ时请备注JSCoreBridge）











