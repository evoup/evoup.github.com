---
layout: post
title: "STL中string到int的类型转换"
date: 2009-06-02 14:04
comments: true
categories:                cplusplus
---


STL的string转int的正确方法不是.data()
应该是

```cpp
aoti(obj.c_str());
```

或者用个别人的string2int函数

```cpp
int string2int(const string &s)
{
   int a;
   sscanf(s.c_str(), "%d", &a);
   return a;
}
```

然后int XX[];
完全可以用vector来实现

```cpp
vector<int > v;
```

v[]一样可以的用的。
 

下面有个类似的

```cpp
int integer(string &s)
{
    int a;
    sscanf(s.c_str(), "%d", &a);
    return a;
}
```

