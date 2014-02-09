---
layout: post
title: "cocos2d-x cocos2d-iphone cocos2d-html5下精灵加载和动画的总结"
date: 2014-02-08 13:10
comments: true
categories: [cocos2d-html5,cocos2d-iphone,cocos2d-html5]
---


加载精灵的几种方法，以下总结下cocos2d-x v2.2.2、cocos2d-iphone v0.99b和cocos2d-html5 v2.2.2三个平台下的创建显示图片和帧动画的方法：

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
//首先定义好资源s_resource
var batchNode = cc.SpriteBatchNode.create(s_resource, 50);
this.addChild(batchNode, 0, TAG_SPRITE_BATCH_NODE);
var batchNodeOld = this.getChildByTag(TAG_SPRITE_BATCH_NODE);
var idx = 0 | (Math.random() * 14);
var x = (idx % 5) * 85;
var y = (0 | (idx / 5)) * 121;
var sprite = cc.Sprite.createWithTexture(batchNodeOld.getTexture(), cc.rect(x, y, 85, 121));
batchNodeOld.addChild(sprite);
{% endcodeblock %}


-------------------------

> <i>batchnode加framecache结合显示多帧动画</i>
{%codeblock lang:objective-c cocos2d-iphone %}
[[CCSpriteFrameCache sharedSpriteFrameCache] addSpriteFramesWithFile:
     @"res/resource.plist"]; //加载resource.plist文件到精灵帧缓冲区
CCSpriteBatchNode *spriteSheet = [CCSpriteBatchNode batchNodeWithFile:@"resources.png"]; //从精灵批处理节点返会一个精灵表
[self addChild:spriteSheet z:1]; //把精灵表加载到self场景对象下，并设置Z轴顺序为1
self.sprite = [CCSprite spriteWithSpriteFrameName:@"resource_00.png"]; //再从缓冲加载单帧图片给精灵
[spriteSheet addChild:sprite]; //精灵表将该精灵呈现出来	
{% endcodeblock %}

{%codeblock lang:javascript cocos2d-html5 %}
var spriteFrameCache = cc.SpriteFrameCache.getInstance();
spriteFrameCache.addSpriteFrames("res/resource.plist","res/resource.png"); //从精灵批处理节点返会一个精灵表
this.aniSprite = cc.Sprite.createWithSpriteFrameName("resource_00.png"); //将精灵表添加到self场景对象
var spritebatch = cc.SpriteBatchNode.create("res/baseResource.png");
spritebatch.addChild(this.aniSprite);
this.addChild(spritebatch);
{% endcodeblock %}




