---
layout: post
title: "五类方法定义javascript类"
date: 2015-04-21 14:20
comments: true
categories: [javascript ]
---
自己写js的时候，总结了类的五种写法，可能还不止，以后增加。
<!--more-->

第一种，this的方式：
{% codeblock lang:javascript %}
    var Test1 = function(){
        var name = "";
        this.setName = function(username){
            name = username;
        }
        this.getName = function(){
            return name;
        }
        this.sayHello = function(){
            return "Hello " + this.getName();
        }
        return this;
    }();
    Test1.setName("penngo");
    console.log("Test1======" + Test1.getName());
    console.log("Test1======" + Test1.sayHello());
{% endcodeblock %}

第二种，return的方式：
{% codeblock lang:javascript %}
    var Test2 = function(){
        var name = "";
        return {
            setName:function(username){
                name = username;
            },
            getName:function(){
                return name;
            },
            sayHello:function(){
                return "Hello " + this.getName();
            }
        };
    }();
    Test2.setName("penngo");
    console.log("Test2======" + Test2.getName());
    console.log("Test2======" + Test2.sayHello());
{% endcodeblock %}

第三种,建立空的构造器函数，使其protoype到一个json结构的对象（这样结构会非常清晰）。最后对空的构造器函数new ， 创建实例：
{% codeblock lang:javascript %}
var pri={
    name:"soso",
    init:function(name){
        this.name=name
    },
    pri:function(){
        alert(this.name)
    }
} 
function _temp(){}
_temp.prototype= pri
var s =new _temp()
s.init(123)
s.pri()
      
var ss=new _temp()
ss.init(1234)
ss.pri()
s.pri()
      
  
var d=new _temp()
d.pri()
{% endcodeblock %}

第四种，jquery扩展类的方式：
{% codeblock lang:javascript %}
var Circle = function(x, y, r) {
      this.x = x;
      this.y = y;
      this.r = r;
}

$.extend(Circle.prototype, {
      area: function() {
            return Math.PI * this.r * this.r;
      },
      diameter: function() {
            return 2 * this.r;
      }
});

var mycircle = new Circle(100, 200, 50);
alert(mycircle.r + ' -> ' + mycircle.area());
{% endcodeblock %}


第五种
{% codeblock lang:javascript %}
var Link = function (){};

Link.prototype = {
    method1: function (){console.log(1);return this;},
    method2: function (){console.log(2);return this;},
    method3: function (){console.log(3);return this;},
    method4: function (){console.log(4);return this;}
}
{% endcodeblock %}

注意第五种的方式，其实就是加了个return this，这样才可以链式调用。
结论：目前先用第五种方式来写，如果需要动态扩展用jq的extend比较推荐。

题外话：不想写class还可以写嵌套function的:)

参考连接：
[http://www.cnblogs.com/jikey/archive/2011/05/13/2045005.html](http://www.cnblogs.com/jikey/archive/2011/05/13/2045005.html)

[http://www.blogjava.net/pengo/archive/2013/01/08/393931.html](http://www.blogjava.net/pengo/archive/2013/01/08/393931.html)

[http://www.cnblogs.com/breakdown/archive/2012/07/18/2557157.html](http://www.cnblogs.com/breakdown/archive/2012/07/18/2557157.html)

[https://forum.jquery.com/topic/creating-a-class-object-with-jquery#14737000000964808](https://forum.jquery.com/topic/creating-a-class-object-with-jquery#14737000000964808)

