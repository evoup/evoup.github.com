---
layout: post
title: "cocos2d-html5-2.2.2初探"
date: 2014-01-23 13:14
comments: true
categories:  [cocos2d-html5]
---

准备用最新的cocos2d-html5 2.2.2的框架作为游戏开发引擎，整个引擎还是逻辑相当清晰好理解。

本文部分参考v2.1.1这篇分析文章
http://blog.csdn.net/honghaier/article/details/8642553

翻了几篇官网几篇文章，发现对应的教程没有同步起来，比如说这篇：
http://www.gamefromscratch.com/post/2012/06/08/Cocos2D-HTML-Tutorial-3All-about-sprites-and-positioning.aspx

<!-- more -->

其间有这么一段代码
```js
s.src = c.engineDir + 'platform/jsloader.js';
```
可以看到在cocos2d目录下没有这个文件，OMG! 淡定缓一缓神，让我们耐心点，这里教你一步一步来掌了解2.2.2版本的api来给原来的教程动手术,我们首先来讲解HelloWorld：

首先看2.2.2的helloworld的代码分布情况：

![Alt text](/images/evoup/cocos2d-html5_2.2.2_code_layout.png)

其中项目必须的文件有cocos2d（cocos2d引擎）、extensions（GUI的扩展）、external（外部引擎如box2d）、lib（cocos2d基本库）。

接下来在根目录下仿照HelloHTML5World中的布局开始做一个项目，创建一个叫做helloworld的文件夹。

看一下运行截图
![Alt text](/images/evoup/cocos2d-html52.2.2helloworld.png)

----------------------

首先是在该文件夹下创建第一个文件index.html
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

这段创建了DOMContentLoaded事件侦听器，干掉全部DOM对象。SingleEngineFile这个参数是如果要把cocos2d这个文件夹下的引擎程序打包成一个文件，则需要指定这个文件的路径。指定好了引擎路径engineDir为../cocos2d这个文件夹，然后加载jsloader.js(这个文件的作用是加载cocos2d需要用到的所有js文件，包括cocos2d.js,main.js,myApp.js),然后把配置文件存到document.ccConfig元素中去。
接着把cocos2d-html5这个id赋到创建到script标签上，然后加载到body标签中。

-------------------

接着创建第三个文件--主程序文件main.js,注意这还不是实现逻辑，而是cocos2d-html5初始化的套路。

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
默认设置fps为1秒60帧
```javascript
// set FPS. the default value is 1.0/60 if you don't call this
director.setAnimationInterval(1.0 / this.config['frameRate']);
```

导演类的replaceScene方法来替换场景为startScene
```javascript
//load resources
cc.LoaderScene.preload(g_resources, function () {
    director.replaceScene(new this.startScene());
}, this);
```

最后实例化app
```javascript
var myApp = new cocos2dApp(HelloWorldScene);
```
-----------------

在正式步入工程文件之前，别急，创建一个src文件夹，然后在该文件夹下创建一个resource.js
```javascript
var s_HelloWorld = "res/HelloWorld.png";
var s_CloseNormal = "res/CloseNormal.png";
var s_CloseSelected = "res/CloseSelected.png";

var g_resources = [
    //image
    {src:s_HelloWorld},
    {src:s_CloseNormal},
    {src:s_CloseSelected}

    //plist

    //fnt

    //tmx

    //bgm

    //effect
];
```
然后在和src相同父目录下创建一个叫做res文件夹，把如下3个文件放入其中：

HelloWorld.png

![Alt text](/images/evoup/cocos2d-html5-res/HelloWorld.png)

CloseNormal.png

![Alt text](/images/evoup/cocos2d-html5-res/CloseNormal.png)

CloseSelected.png

![Alt text](/images/evoup/cocos2d-html5-res/CloseSelected.png)

这就完成了资源文件的定义和提供。

-----------------

终于到工程主文件了，在src下创建一个叫做myApp.js的文件
```javascript
var Helloworld = cc.Layer.extend({
    isMouseDown:false,
    helloImg:null,
    helloLabel:null,
    circle:null,
    sprite:null,

    init:function () {
        //////////////////////////////
        // 1. super init first
        this._super();

        /////////////////////////////
        // 2. add a menu item with "X" image, which is clicked to quit the program
        //    you may modify it.
        // ask director the window size
        var size = cc.Director.getInstance().getWinSize();

        // add a "close" icon to exit the progress. it's an autorelease object
        var closeItem = cc.MenuItemImage.create(
            "res/CloseNormal.png",
            "res/CloseSelected.png",
            function () {
                history.go(-1);
            },this);
        closeItem.setAnchorPoint(0.5, 0.5);

        var menu = cc.Menu.create(closeItem);
        menu.setPosition(0,0);
        this.addChild(menu, 1);
        closeItem.setPosition(size.width - 20, 20);

        /////////////////////////////
        // 3. add your codes below...
        // add a label shows "Hello World"
        // create and initialize a label
        this.helloLabel = cc.LabelTTF.create("Hello World", "Arial", 38);
        // position the label on the center of the screen
        this.helloLabel.setPosition(size.width / 2, 0);
        // add the label as a child to this layer
        this.addChild(this.helloLabel, 5);

        var lazyLayer = cc.Layer.create();
        this.addChild(lazyLayer);

        // add "HelloWorld" splash screen"
        this.sprite = cc.Sprite.create("res/HelloWorld.png");
        this.sprite.setPosition(size.width / 2, size.height / 2);
        this.sprite.setScale(0.5);
        this.sprite.setRotation(180);

        lazyLayer.addChild(this.sprite, 0);

        var rotateToA = cc.RotateTo.create(2, 0);
        var scaleToA = cc.ScaleTo.create(2, 1, 1);

        this.sprite.runAction(cc.Sequence.create(rotateToA, scaleToA));
        this.helloLabel.runAction(cc.Spawn.create(cc.MoveBy.create(2.5, cc.p(0, size.height - 40)),cc.TintTo.create(2.5,255,125,0)));

        this.setTouchEnabled(true);
        return true;
    },
    // a selector callback
    menuCloseCallback:function (sender) {
        cc.Director.getInstance().end();
    },
    onTouchesBegan:function (touches, event) {
        this.isMouseDown = true;
    },
    onTouchesMoved:function (touches, event) {
        if (this.isMouseDown) {
            if (touches) {
                //this.circle.setPosition(touches[0].getLocation().x, touches[0].getLocation().y);
            }
        }
    },
    onTouchesEnded:function (touches, event) {
        this.isMouseDown = false;
    },
    onTouchesCancelled:function (touches, event) {
        console.log("onTouchesCancelled");
    }
});

var HelloWorldScene = cc.Scene.extend({
    onEnter:function () {
        this._super();
        var layer = new Helloworld();
        layer.init();
        this.addChild(layer);
    }
});
```

####分析
```javascript
isMouseDown:false,
helloImg:null,
helloLabel:null,
circle:null,
sprite:null,
```
这是对接下来用到的成员变量的定义

再看接下来的代码
```javascript
    init:function () {
        //首先初始化超类
        this._super();

        //获取屏宽
        var size = cc.Director.getInstance().getWinSize();

        //添加一个关闭按钮到菜单项
        var closeItem = cc.MenuItemImage.create(
            "res/CloseNormal.png",
            "res/CloseSelected.png",
            function () {
                history.go(-1);
            },this);
        //设置锚点，默认就是0.5，0.5
        closeItem.setAnchorPoint(0.5, 0.5);
        //创建关闭按钮
        var menu = cc.Menu.create(closeItem);
        //设置菜单项位置
        menu.setPosition(0,0);
        //添加菜单项到当前层上
        this.addChild(menu, 1);
        //设置菜单的位置为屏幕右下角
        closeItem.setPosition(size.width - 20, 20);

        //创建一个叫做helloworld的文本标签
        this.helloLabel = cc.LabelTTF.create("Hello World", "Arial", 38);
        //把文本标签放到屏幕中央位置
        this.helloLabel.setPosition(size.width / 2, 0);
        //添加该文本标签放到当前层上
        this.addChild(this.helloLabel, 5);
        //创建一个新层lazylay并放入当前层
        var lazyLayer = cc.Layer.create();
        this.addChild(lazyLayer);

        // 添加一个能旋转的helloworld精灵
        this.sprite = cc.Sprite.create("res/HelloWorld.png");
        this.sprite.setPosition(size.width / 2, size.height / 2);
        //先缩小50%且倒过来
        this.sprite.setScale(0.5);
        this.sprite.setRotation(180);
        //添加到lazyLayer层
        lazyLayer.addChild(this.sprite, 0);
        //定义2个动作分别为旋转2秒到0度和缩放2秒无x轴y轴缩放
        var rotateToA = cc.RotateTo.create(2, 0);
        var scaleToA = cc.ScaleTo.create(2, 1, 1);
        //让helloworld精灵依次运行这2个动作
        this.sprite.runAction(cc.Sequence.create(rotateToA, scaleToA));
        //使文本标签同时运行ccMoveBy动作（后面讲）和CCTintTo动作（使文字）
        this.helloLabel.runAction(cc.Spawn.create(cc.MoveBy.create(2.5, cc.p(0, size.height - 40)),cc.TintTo.create(2.5,255,125,0)));
        //设置为支持触碰
        this.setTouchEnabled(true);
        return true;
    },
```
需要注意的是
1) cc.Sequence代表依次运行动作，而cc.Spawn代表同时运行动作
2) CCMoveTo是“移动到这里”；而CCMoveBy则是“相对于之前点再移动”，比如说说这里需要两个坐标pos1（x1，y1），pos2（x2，y2），用CCMoveTo的话，就是将对象由pos1移动到pos2，而CCMOveBy则是对象的终坐标是在pos1的基础上再加上（矢量相加）pos2，终坐标pos3=pos1+pos2。

接下来是一些触碰事件的回调函数
```javascript
    //点击关闭按钮的回调函数，退出实例
    menuCloseCallback:function (sender) {
        cc.Director.getInstance().end();
    },
    //按下按钮的状态
    onTouchesBegan:function (touches, event) {
        this.isMouseDown = true;
    },
    //按下按钮移动时
    onTouchesMoved:function (touches, event) {
        if (this.isMouseDown) {
            if (touches) {
                //this.circle.setPosition(touches[0].getLocation().x, touches[0].getLocation().y);
            }
        }
    },
    //放开按钮的状态
    onTouchesEnded:function (touches, event) {
        this.isMouseDown = false;
    },
    //离开按钮的状态
    onTouchesCancelled:function (touches, event) {
        console.log("onTouchesCancelled");
    }
```

最后

```javascript
var HelloWorldScene = cc.Scene.extend({
    onEnter:function () {
        this._super();
        var layer = new Helloworld();
        layer.init();
        this.addChild(layer);
    }
});
```


进入了之后就把这个layer再加载到场景上，初步分析终了，等完成一个项目之后再来接着写点东西。





