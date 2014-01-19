---
layout: post
title: "android webview helloworld分析"
date: 2014-01-19 17:05
comments: true
categories: [android]
---

###加载示例代码
官方示例代码
https://code.google.com/p/apps-for-android/source/browse/#git%2FSamples%2FWebViewDemo

运行Eclipse(本文提到的eclipse版本是Eclipse ide for java developer KEPLER)，选择File->Import...->Android->Existing Android Code Into Workspace,Next，在Root Directory中选择解压的目录下的Samples文件夹下的WebViewDemo。

如果直接run，会出现错误` Project has no project.properties file! Edit the project properties to set one.`。
这时只要右键项目属性properties -> android -> Project Build Target 把里面的Androidx.x改为对应的google APIs版本。google APIs是android apis的超集，Android只是其中一个用于移动设备的OS，可以有限的调用一些google api。

###分析app部分代码
请看示例java代码
<!-- more -->

{% codeblock lang:java WebViewDemo.java %}
package com.google.android.webviewdemo;

import android.app.Activity;
import android.os.Bundle;
import android.os.Handler;
import android.util.Log;
import android.webkit.JsResult;
import android.webkit.WebChromeClient;
import android.webkit.WebSettings;
import android.webkit.WebView;

/**
 * Demonstrates how to embed a WebView in your activity. Also demonstrates how
 * to have javascript in the WebView call into the activity, and how the activity 
 * can invoke javascript.
 * <p>
 * In this example, clicking on the android in the WebView will result in a call into
 * the activities code in {@link DemoJavaScriptInterface#clickOnAndroid()}. This code
 * will turn around and invoke javascript using the {@link WebView#loadUrl(String)}
 * method.
 * <p>
 * Obviously all of this could have been accomplished without calling into the activity
 * and then back into javascript, but this code is intended to show how to set up the 
 * code paths for this sort of communication.
 *
 */
public class WebViewDemo extends Activity {

    private static final String LOG_TAG = "WebViewDemo";

    private WebView mWebView;

    private Handler mHandler = new Handler();

    @Override
    public void onCreate(Bundle icicle) {
        super.onCreate(icicle);
        setContentView(R.layout.main);
        mWebView = (WebView) findViewById(R.id.webview);

        WebSettings webSettings = mWebView.getSettings();
        webSettings.setSavePassword(false);
        webSettings.setSaveFormData(false);
        webSettings.setJavaScriptEnabled(true);
        webSettings.setSupportZoom(false);

        mWebView.setWebChromeClient(new MyWebChromeClient());

        mWebView.addJavascriptInterface(new DemoJavaScriptInterface(), "demo");

        mWebView.loadUrl("file:///android_asset/demo.html");
    }

    final class DemoJavaScriptInterface {

        DemoJavaScriptInterface() {
        }

        /**
         * This is not called on the UI thread. Post a runnable to invoke
         * loadUrl on the UI thread.
         */
        public void clickOnAndroid() {
            mHandler.post(new Runnable() {
                public void run() {
                    mWebView.loadUrl("javascript:wave()");
                }
            });

        }
    }

    /**
     * Provides a hook for calling "alert" from javascript. Useful for
     * debugging your javascript.
     */
    final class MyWebChromeClient extends WebChromeClient {
        @Override
        public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
            Log.d(LOG_TAG, message);
            result.confirm();
            return true;
        }
    }
}
{% endcodeblock %}

首先需要搞懂Activity这个核心概念，基本上是进行android的活动对象事件调度的类，我们看一下android api中的activity的生命周期：

![Alt text](/images/evoup/activity_lifecycle.png)

它有以下7个方法可以被重写：
```java
public class Activity extends ApplicationContext {  
       protected void onCreate(Bundle savedInstanceState);  
       protected void onStart();     
       protected void onRestart();  
       protected void onResume();  
       protected void onPause();   
       protected void onStop();  
       protected void onDestroy();  
   }  
```
也就是onCreate创建，onStart开始、onRestart重启、onResume恢复、onPause暂停、onStop停止和onDestory销毁时，这些方法都能够被重写，不难看出这是方法和生命周期是息息相关的。
本例子就只要重写onCreate即可，看代码：
```java
    @Override
    public void onCreate(Bundle icicle) {
        super.onCreate(icicle);
        setContentView(R.layout.main);
        mWebView = (WebView) findViewById(R.id.webview);

        WebSettings webSettings = mWebView.getSettings();
        webSettings.setSavePassword(false);
		
        webSettings.setSaveFormData(false);
        webSettings.setJavaScriptEnabled(true);
        webSettings.setSupportZoom(false);

        mWebView.setWebChromeClient(new MyWebChromeClient());

        mWebView.addJavascriptInterface(new DemoJavaScriptInterface(), "demo");

        mWebView.loadUrl("file:///android_asset/demo.html");
    }
```
其中R.java这个为android的资源类，工程自动会维护这里暂时不用去考虑。首先是调用超类的onCreate方法来必要的初始化。然后设置试图为资源类的main。再来从资源类的webview中找到Id返回WebView类型的对象mWebView这个变量中。然后是WebSettings类对象webSettings的参数设置，这里setSavePassword和setSaveFormData代表不要保存表单控件的数据和密码，setJavaScriptEnabled设置为允许webview调用JavaScript，setSupportZoom设置为不允许缩放。由于要用到JavaScript，所以需要调用setWebChromeClient。

我们需要仔细看下addJavascriptInterface这个函数的说明

> Injects the supplied Java object into this WebView. The object is injected into the JavaScript context of the main frame, using the supplied name. This allows the Java object's methods to be accessed from JavaScript. For applications targeted to API level JELLY_BEAN_MR1 and above, only public methods that are annotated with JavascriptInterface can be accessed from JavaScript. For applications targeted to API level JELLY_BEAN or below, all public methods (including the inherited ones) can be accessed, see the important security note below for implications.

我给翻译了下：它注入提供的Java对象到这个WebView。对象使用提供的名字注入JavaScript上下文的主要框架。这允许从JavaScript访问Java对象的方法。针对应用程序API级别在JELLY_BEAN_MR1以上的,只有JavascriptInterface的公共方法与注释可以从JavaScript访问。针对应用程序API级别或低于JELLY_BEAN的,所有公共方法(包括继承的)可以访问,参见下面重要的安全注意影响。

再看下2个参数，前一个为注入webview的Java对象（DemoJavaScriptInterface），后一个为暴露给JavaScript的对象名字（demo），本例中除了webview把点击消息传递给html，html中的JavaScript也要和webview交互，在哪里交互？伟大的google为我们想到JavaScript的顶层window对象，在下面动态创建了一个子对象，这里就是第二个参数demo，这个等等到了html再讲。

马上来看第一个参数的实现
```java
    final class DemoJavaScriptInterface {

        DemoJavaScriptInterface() {
        }

        /**
         * This is not called on the UI thread. Post a runnable to invoke
         * loadUrl on the UI thread.
         */
        public void clickOnAndroid() {
            mHandler.post(new Runnable() {
                public void run() {
                    mWebView.loadUrl("javascript:wave()");
                }
            });

        }
    }
```
直接一个构造函数什么都不做，然后直接用句柄提交实现了Runnable类的引用，再深入Runnable探究下

> The Runnable interface should be implemented by any class whose instances are intended to be executed by a thread. The class must define a method of no arguments called run. 

Runnable就一个方法，run给实现了就行，这里是直接调用demo.html页面里的JavaScript的wave方法。

看下一段代码
```java
    /**
     * Provides a hook for calling "alert" from javascript. Useful for
     * debugging your javascript.
     */
    final class MyWebChromeClient extends WebChromeClient {
        @Override
        public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
            Log.d(LOG_TAG, message);
            result.confirm();
            return true;
        }
    }
```
这里是对刚才指定的WebChromeClient对象MyWebChromeClient，进行onJsAlert的重写，可以看到其实就是在打印日志。

--------------------------------

###分析html部分代码
app代码完了，再来看html
{% codeblock lang:html demo.html %}
<html>
	<script language="javascript">
	    /* This function is invoked by the activity */
		function wave() {
		    alert("1");
			document.getElementById("droid").src="android_waving.png";
			alert("2");
		}
	</script>
	<body>
	    <!-- Calls into the javascript interface for the activity -->
	    <a onClick="window.demo.clickOnAndroid()"><div style="width:80px;
			margin:0px auto;
			padding:10px;
			text-align:center;
			border:2px solid #202020;" >
				<img id="droid" src="android_normal.png"/><br>
				Click me!
		</div></a>
	</body>
</html>
{% endcodeblock %}

注意看window.demo.clickOnAndroid这个JavaScript方法，刚刚我们在app代码里用addJavascriptInterface方法生成好了一个window.demo对象，就是通过这个对象，就可以调用app内置方法来实现网页的跳转。


####小结
到此程序分析完毕,整个交互过程还是比较容易理解，就是先继承一个Activity类，关联好html页面，实现onCreate方法，在其中指定好app方法以及为javascript暴露对象名，然后进入html调用暴露的对象名及其方法进行跳转。

-----------------------


最后，打开应用程序，它看起来应该像这样:)

![Alt text](/images/evoup/android_webview_sample.png)

