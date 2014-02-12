---
layout: post
title: "cocos2d-x cocos2d-iphone cocos2d-html5下加载精灵的总结"
date: 2014-02-08 13:10
comments: true
categories: [cocos2d-x,cocos2d-iphone,cocos2d-html5]
---


加载精灵的几种套路，以下总结下cocos2d-x v2.2.2、cocos2d-iphone v0.99b和cocos2d-html5 v2.2.2三个平台下的创建显示图片和帧动画的方法：

<!-- more -->

> <i>直接从图片创建：</i>
{%codeblock lang:cpp cocos2d-x %}
CCSprite *sprite=CCSprite::create("res/pic.png"); //从pic.png创建一个精灵
sprite->setPosition(ccp(100, 240)); //设置坐标为x=100,y=240
sprite->setScale(0.5); //缩小50%
this->addChild(sprite,0); //加载到this场景对象下
{% endcodeblock %}

{%codeblock lang:objective-c cocos2d-iphone %}
CCSprite *sprite = [CCSprite spriteWithFile:@"res/pic.png"]; //从pic.png创建一个精灵
sprite.position = ccp(100, 240); //设置坐标为x=100,y=240
sprite.scale = 0.5; //缩小50%
[self addChild:sprite z:0]; //加载到self场景对象下
{% endcodeblock %}

{%codeblock lang:javascript cocos2d-html5 %}
var sprite; //定义sprite变量
this.sprite = cc.Sprite.create("res/pic.png"); //从pic.png创建一个精灵
this.sprite.setPosition(new cc.Point(100,240)); //设置坐标为x=100,y=240
this.sprite.scale = 0.5; //缩小50%
this.addChild(this.sprite, 0); //加载到this场景对象下
{% endcodeblock %}

-------------------------

> <i>batchnode显示大量精灵：</i>
> <br />注意：batchnode的限制是所有添加到它里面的CCSprite节点的Z轴顺序必须相同，且须使用相同纹理。

{%codeblock lang:cpp cocos2d-x %}
CCSpriteBatchNode * batchNode = CCSpriteBatchNode::create("pic.png"); //创建批渲染对象
this->addChild(batchNode); //批渲染对象加载到self场景对象下
for (int i=0;i<100;i++) {
    CCSprite *sprite = CCSprite::create("pic.png"); //载入图片到精灵，方法效率看起来可以优化（code来自《cocos2d-x游戏开发之旅》）
	batchNode->addChild(sprite); //将精灵添加到批渲染对象
}
{% endcodeblock %}

{%codeblock lang:objective-c cocos2d-iphone %}
CCSpriteBatchNode * batchNode = [CCSpriteBatchNode batchNodeWithFile:
    @"pic.png"]; //创建批渲染对象
[self addChild:batch]; //批渲染对象加载到self场景对象下
for (int i=0;i<100;i++) {
    CCSprite *sprite = [CCSpriteBatchNode batchNodeWithFile:@"pic.png"]; //载入图片到精灵
	[batchNode addChild:sprite]; //将精灵添加到批渲染对象
}	
{% endcodeblock %}

{%codeblock lang:javascript cocos2d-html5 %}
//首先定义好资源
var batchNode = cc.SpriteBatchNode.create("pic.png", 50); //定义批渲染对象
this.addChild(batchNode,0); //把批渲染节点加到this场景对象
for (i=0;i<100;i++) { //载入100个精灵并分别加到批渲染节点
    var sprite = cc.Sprite.createWithTexture(batchNode.getTexture(), cc.rect(0, 0, 85, 121));
    batchNode.addChild(sprite);
}
{% endcodeblock %}


-------------------------

> <i>batchnode加framecache结合显示单帧图片</i>
{%codeblock lang:cpp cocos2d-x %}
CCSpriteBatchNode *batchNode = CCSpriteBatchNode::create("resource.png"); //创建批渲染对象
this->addChild(batchNode); //批渲染对象加载到this场景对象下
CCSpriteFrameCache::sharedSpriteFrameCache()->addSpriteFramesWithFile("resources.plist"); //帧缓冲加入plist打包好的文件，用texture packer软件可以打包
CCSprite * sprite = CCSprite::createWithSpriteFrameName("resources_00.png"); //读取单帧到精灵
batchNode->addChild(sprite); //加到批渲染对象
{% endcodeblock %}

{%codeblock lang:objective-c cocos2d-iphone %}
[[CCSpriteFrameCache sharedSpriteFrameCache] addSpriteFramesWithFile:
     @"res/resources.plist"]; //加载resource.plist文件到精灵帧缓冲区
CCSpriteBatchNode *spriteSheet = [CCSpriteBatchNode batchNodeWithFile:@"resources.png"]; //从精灵批渲染节点返会一个精灵表
[self addChild:spriteSheet z:1]; //把精灵表加载到self场景对象下，并设置Z轴顺序为1
self.sprite = [CCSprite spriteWithSpriteFrameName:@"resource_00.png"]; //再从缓冲加载单帧图片给精灵
[spriteSheet addChild:sprite]; //精灵表将该精灵呈现出来	
{% endcodeblock %}

{%codeblock lang:javascript cocos2d-html5 %}
var spriteFrameCache = cc.SpriteFrameCache.getInstance();
spriteFrameCache.addSpriteFrames("res/resources.plist","res/resources.png"); //从精灵批渲染节点返会一个精灵表
this.sprite = cc.Sprite.createWithSpriteFrameName("resource_00.png"); //将单帧精灵图片添加到this场景对象
var batchNode = cc.SpriteBatchNode.create("res/resources.png");
batchNode.addChild(this.sprite);
this.addChild(batchNode);
{% endcodeblock %}




