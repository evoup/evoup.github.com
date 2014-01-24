---
layout: post
title: "cocos2d-html5-2.2.2初探"
date: 2014-01-23 13:14
comments: true
categories:  [cocos2d-html5]
---

准备用最新的cocos2d-html5 2.2.2的框架作为游戏开发引擎，整个引擎还是逻辑相当清晰好理解。

翻了几篇官网几篇文章，发现对应的教程没有同步起来，比如说这篇：
http://www.gamefromscratch.com/post/2012/06/08/Cocos2D-HTML-Tutorial-3All-about-sprites-and-positioning.aspx

<!-- more -->

其间有这么一段代码
```js
s.src = c.engineDir + 'platform/jsloader.js';
```
可以看到在cocos2d目录下没有这个文件，OMG! 淡定缓一缓神，让我们耐心点，这里教你一步一步来改写,我们首先来讲解HelloWorld：

首先看2.2.2的helloworld的代码分布情况：

![Alt text](/images/evoup/cocos2d-html5_2.2.2_code_layout.png)

其中项目必须的文件有cocos2d（cocos2d引擎）、extensions（GUI的扩展）、external（外部引擎如box2d）、lib（cocos2d基本库）。

接下来在根目录下仿照HelloHTML5World中的布局开始做一个项目，就叫做helloworld吧。
首先是index.html
{% codeblock lang:html index.html %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>hello cocos2d-html5</title>
    <link rel="icon" type="iamge/GIF" href="res/favicon.ico"/>
    <meta name="viewport" content="user-scalable=no" />
    <meta name="apple-mobile-web-app-capable" content="yes" />
    <meta name="full-screen" content="yes" />
    <meta name="screen-orientation" content="portrait" />
    <meta name="x5-fullscreen" content="true" />
    <meta name="360-fullscreen" content="true" />
    <style>
        body, canvas, div {
            -moz-user-select: none;
            -webkit-user-select: none;
            -ms-user-select: none;
            -khtml-user-select: none;
            -webkit-tap-highlight-color: rgba(0,0,0,0);
        }
    </style>
</head>
<body style="padding:0; margin: 0; background: #000;">
<canvas id="gameCanvas" width="800" height="450"></canvas>
<script src="cocos2d.js"></script>
</body>
</html>
{% endcodeblock %}

####解释:
```html
<meta name="viewport" content="user-scalable=no" />
```

viewport这个是移动端的meta设置，content设置为user-scalable=no代表在移动设备上 不可缩放。

```html
<meta name="apple-mobile-web-app-capable" content="yes" />
```

apple-mobile-web-app-capable这个是移动端的参数，全屏模式下content设置为yes，否则不用设置。可以通过window.navigator.standalone属性来得知是否网页是否处于全屏模式。

```html
<meta name="full-screen" content="yes" />
```

这个是给某些浏览器的全屏设置，比如说ucweb

```html
<meta name="screen-orientation" content="portrait" />
```
screen-orientation这个meta参数代表移动端的屏幕朝向，有2个portrait和landscape，分别代表竖向和横向,这portrait是竖向了。

```html
<meta name="x5-fullscreen" content="true" />
```
这个是腾讯qq浏览器的全屏

```html
<meta name="360-fullscreen" content="true" />
```
360浏览器的全屏

```html
<style>
    body, canvas, div {
        -moz-user-select: none;
        -webkit-user-select: none;
        -ms-user-select: none;
        -khtml-user-select: none;
        -webkit-tap-highlight-color: rgba(0,0,0,0);
    }
</style>
```
其中-moz-user-select、-webkit-user-select、-ms-user-select和-khtml-user-select都取值none，代表文字都不选中。而-webkit-tap-highlight-color: rgba(0,0,0,0);其中rgba的最后一个参数0代表点击一个链接或者通过JavaScript定义可点击元素的时候禁止出现灰色背景。
最后从同域引入cocos2d.js。


-------------------

接着创建第二个文件cocos2d.js
{% codeblock lang:js cocos2d.js %}
(function () {
    var d = document;
    var c = {
        COCOS2D_DEBUG:2, //0 to turn debug off, 1 for basic debug, and 2 for full debug
        box2d:false,
        chipmunk:false,
        showFPS:true,
        frameRate:60,
        loadExtension:false,
        renderMode:0,       //Choose of RenderMode: 0(default), 1(Canvas only), 2(WebGL only)
        tag:'gameCanvas', //the dom element to run cocos2d on
        engineDir:'../cocos2d/',
        //SingleEngineFile:'',
        appFiles:[
            'src/resource.js',
            'src/myApp.js'//add your own files in order here
        ]
    };

    if(!d.createElement('canvas').getContext){
        var s = d.createElement('div');
        s.innerHTML = '<h2>Your browser does not support HTML5 canvas!</h2>' +
            '<p>Google Chrome is a browser that combines a minimal design with sophisticated technology to make the web faster, safer, and easier.Click the logo to download.</p>' +
            '<a href="http://www.google.com/chrome" target="_blank"><img src="http://www.google.com/intl/zh-CN/chrome/assets/common/images/chrome_logo_2x.png" border="0"/></a>';
        var p = d.getElementById(c.tag).parentNode;
        p.style.background = 'none';
        p.style.border = 'none';
        p.insertBefore(s,d.getElementById(c.tag));

        d.body.style.background = '#ffffff';
        return;
    }

    window.addEventListener('DOMContentLoaded', function () {
        this.removeEventListener('DOMContentLoaded', arguments.callee, false);
        //first load engine file if specified
        var s = d.createElement('script');
        /*********Delete this section if you have packed all files into one*******/
        if (c.SingleEngineFile && !c.engineDir) {
            s.src = c.SingleEngineFile;
        }
        else if (c.engineDir && !c.SingleEngineFile) {
            s.src = c.engineDir + 'jsloader.js';
        }
        else {
            alert('You must specify either the single engine file OR the engine directory in "cocos2d.js"');
        }
        /*********Delete this section if you have packed all files into one*******/

            //s.src = 'Packed_Release_File.js'; //IMPORTANT: Un-comment this line if you have packed all files into one

        document.ccConfig = c;
        s.id = 'cocos2d-html5';
        d.body.appendChild(s);
        //else if single file specified, load singlefile
    });
})();
{% endcodeblock %}

####解释:
这整个就是一个匿名函数,然后调用的写法。

首先创建一个局部变量d，赋值为document
```js
var d = document;
```

然后创建一个c对象，提供全部配置信息
```js
var c = {
    COCOS2D_DEBUG:2, //0 to turn debug off, 1 for basic debug, and 2 for full debug
    box2d:false,
    chipmunk:false,
    showFPS:true,
    frameRate:60,
    loadExtension:false,
    renderMode:0,       //Choose of RenderMode: 0(default), 1(Canvas only), 2(WebGL only)
    tag:'gameCanvas', //the dom element to run cocos2d on
    engineDir:'../cocos2d/',
    //SingleEngineFile:'',
    appFiles:[
        'src/resource.js',
        'src/myApp.js'//add your own files in order here
    ]
};
```

其中:
COCOS2D_DEBUG为调试等级（0关闭，1基本，2完全）；
box2d为是否采用box2d物理引擎，这里不使用设置为no；
chipmunk为一款物理引擎，同样不使用；
showFPS是否显示FPS，是；
loadExtension是否加载扩展，否；
renderMode渲染，0为默认，1为仅Canvas方式，2为仅WebGL方式;
tag为运行cocos2d的dom元素;
engineDir为cocos2d的引擎目录;
appFiles为一个列表，目前存2个文件的地址，src/resource.js和src/myApp.js;


下面这段为html5探测代码，如果探测失败，会出出现一个google的chrome浏览器的logo
```js
if(!d.createElement('canvas').getContext){
    var s = d.createElement('div');
    s.innerHTML = '<h2>Your browser does not support HTML5 canvas!</h2>' +
        '<p>Google Chrome is a browser that combines a minimal design with sophisticated technology to make the web faster, safer, and easier.Click the logo to download.</p>' +
        '<a href="http://www.google.com/chrome" target="_blank"><img src="http://www.google.com/intl/zh-CN/chrome/assets/common/images/chrome_logo_2x.png" border="0"/></a>';
    var p = d.getElementById(c.tag).parentNode;
    p.style.background = 'none';
    p.style.border = 'none';
    p.insertBefore(s,d.getElementById(c.tag));

    d.body.style.background = '#ffffff';
    return;
}
```

再看下一段
```js
window.addEventListener('DOMContentLoaded', function () {
    this.removeEventListener('DOMContentLoaded', arguments.callee, false);
    //first load engine file if specified
    var s = d.createElement('script');
    /*********Delete this section if you have packed all files into one*******/
    if (c.SingleEngineFile && !c.engineDir) {
    s.src = c.SingleEngineFile;
    }
    else if (c.engineDir && !c.SingleEngineFile) {
    s.src = c.engineDir + 'jsloader.js';
    }
    else {
    alert('You must specify either the single engine file OR the engine directory in "cocos2d.js"');
    }
    /*********Delete this section if you have packed all files into one*******/

    //s.src = 'Packed_Release_File.js'; //IMPORTANT: Un-comment this line if you have packed all files into one

    document.ccConfig = c;
    s.id = 'cocos2d-html5';
    d.body.appendChild(s);
    //else if single file specified, load singlefile
});
```

这段创建了DOMContentLoaded事件侦听器，干掉全部DOM对象。SingleEngineFile这个参数是如果要把cocos2d这个文件夹下的引擎程序打包成一个文件，则需要指定这个文件的路径。指定好了引擎路径engineDir为../cocos2d这个文件夹，然后加载jsloader.js,然后把配置文件存到document.ccConfig元素中去。
接着把cocos2d-html5这个id赋到创建到script标签上，然后加载到body标签中。

-------------------

接着创建主程序文件main.js,注意这还不是实现逻辑，而是cocos2d-html5初始化的套路。

{% codeblock lang:javascript main.js %}
var cocos2dApp = cc.Application.extend({
    config:document['ccConfig'],
    ctor:function (scene) {
        this._super();
        this.startScene = scene;
        cc.COCOS2D_DEBUG = this.config['COCOS2D_DEBUG'];
        cc.initDebugSetting();
        cc.setup(this.config['tag']);
        cc.AppController.shareAppController().didFinishLaunchingWithOptions();
    },
    applicationDidFinishLaunching:function () {
        if(cc.RenderDoesnotSupport()){
            //show Information to user
            alert("Browser doesn't support WebGL");
            return false;
        }
        // initialize director
        var director = cc.Director.getInstance();

        cc.EGLView.getInstance().resizeWithBrowserSize(true);
        cc.EGLView.getInstance().setDesignResolutionSize(800, 450, cc.RESOLUTION_POLICY.SHOW_ALL);

        // turn on display FPS
        director.setDisplayStats(this.config['showFPS']);

        // set FPS. the default value is 1.0/60 if you don't call this
        director.setAnimationInterval(1.0 / this.config['frameRate']);

        //load resources
        cc.LoaderScene.preload(g_resources, function () {
            director.replaceScene(new this.startScene());
        }, this);

        return true;
    }
});
var myApp = new cocos2dApp(HelloWorldScene);
{% endcodeblock %}

主要的功能都在cc.Application.extend中实现。首先config配置参数为document['ccConfig']
然后看ctor：（顾名思义：config to run 配置转为启动）以scene为参数的匿名函数，先调用_super()方法这个引擎方法进行初始化，然后指定startScene开始场景为scene。调试等级cc.COCOS2D_DEBUG为cocos2d.js中指定的调试等级，cc.initDebugSetting为初始化调试设置，cc.setup(this.config['tag'])在名为tag的dom对象上创建cavas,cc.AppController.shareAppController().didFinishLaunchingWithOptions();这个是仿照cocos2d-iphone的函数命名的启动函数（顾名思义：就是启动结束会加载的函数）。

在正式进入写代码之前，稍等再看一下applicationDidFinishLaunching这个函数

先检查渲染是否支持，不支持就弹出“Browser doesn`t support WebGL”

```javascript
if(cc.RenderDoesnotSupport()){
    //show Information to user
    alert("Browser doesn't support WebGL");
    return false;
}
```

初始化director
```javascript
// initialize director
var director = cc.Director.getInstance();
```

重置浏览器大小
```javascript
cc.EGLView.getInstance().resizeWithBrowserSize(true);
cc.EGLView.getInstance().setDesignResolutionSize(800, 450, cc.RESOLUTION_POLICY.SHOW_ALL);
```
打开显卡FPS
```javascript
// turn on display FPS
director.setDisplayStats(this.config['showFPS']);
```





