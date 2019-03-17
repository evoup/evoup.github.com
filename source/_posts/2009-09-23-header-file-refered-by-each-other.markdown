---
layout: post
title: "关于头文件互相包含的解决方法"
date: 2009-09-23 23:09
comments: true
categories:     cplusplus
---
` 写在前面，如果你这么写了，基本上不是很好的写法，以下是我的惨痛经验教训。 `

我的游戏项目中有2个文件，其中cls_player.h包含cls_enemy.h,而且cls_enemy.h也要cls_player.h，这样子一来就成了重复包含了，虽然用防止重复包含头文件的宏#ifndef #define #endif，还是会报错。解决的方法初步看起来如下：

<!-- more -->

<hr>
不要在.h里互相包含头文件.

可以在另一个A的.h里声明一下class B,然后用B的指针就可以.包头文件放在A的cpp里包含就可以了.
</hr>
声明的时候只要在cls_enemy.h里面加上一句class class_player;就可以了。这样cls_enemy.h里就不要去包含cls_player.h了，而cls_player.h则照旧包含cls_enemy.h。但是在设计的时候发现还不行。

总结，如果2个类是差不多的类，最好放在一个文件里。

java倒是没有什么问题，所以java的话会这样子cls_player.java和cls_enemy.java
这样子大概会这么写

{% codeblock lang:java cls_player.java %}
package com.evoup.player_enemy
import ...
class cls_player{
   ...
}
{% endcodeblock %}

然后看搜到了这篇文章
http://topic.csdn.net/t/20051106/18/4375160.html

http://hi.baidu.com/030502505/blog/item/4a7eaba2e9cd12aacaefd06f.html/cmtid/11994e2a68a059315243c1b4


<hr>
##正解
现在我总结一下问题的解决过程和方法：

 
方法一：利用友元类
 
我一共有两个类，由于要在两个类的头文件里互相应用对方，所以，在每一个类的头文件里面现包含另一个类的头文件，然后在该类的定义中声明另一个类为友元类。如下：

{% codeblock lang:cpp %}
#include "B.h"
 
class CA: public CDialog
{
    friend class CB;
    public:
    CB* m_b; //注意一定要是指针类型
}
{% endcodeblock %}
 
在另一个类中可以这样声明:

{% codeblock lang:cpp %}
#include "A.h"
class CB: public CDialog
{
    friend class CA;
    public:
    CA * m_a; //注意一定要是指针类型
}
{% endcodeblock %}
 
最后关键的是在每一个类的构造函数里 new 一个对方的类出来就ok了！
 
方法二：
 
我一共有两个类，由于要在两个类的头文件里互相应用对方，所以，在每一个类的头文件里面现包含另一个类的头文件，然后在该类的定义中声明另一个类为友元类。如下：

{% codeblock lang:cpp %}
#include "B.h"
class CA: public CDialog
{
    friend class CB;
    public:
    CB* m_b; //注意一定要是指针类型
}
{% endcodeblock %}
 
在另一个类中可以这样声明:

{% codeblock lang:cpp %}
class CA;
class CB: public CDialog
{
    public:
    CA * m_a; //注意一定要是指针类型
}
{% endcodeblock %}
 
在cb.cpp文件中包含头文件

{% codeblock lang:cpp %}
#include "ca.h"
{% endcodeblock %}
 
最后关键的是在每一个类的构造函数里 new 一个对方的类出来就ok了
