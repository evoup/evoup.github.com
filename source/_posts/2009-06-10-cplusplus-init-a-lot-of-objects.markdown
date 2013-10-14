---
layout: post
title: "初始化一组C++对象"
date: 2009-06-10 13:29
comments: true
categories:   cplusplus
---

今天要初始化一组对象

```cpp
cls_enemy * enemy0[]={
new cls_enemy(),
new cls_enemy(),
new cls_enemy(),
new cls_enemy(),
new cls_enemy(),
new cls_enemy(),
new cls_enemy()
};
```

像这样初始化也太麻烦了吧，看看有什么对象链表之类的东西。

在VC里可以用CStringArray和CStringList或者CPtrArray或者CPtrList来做，那么STL里是一样的。正确做法可以是push_back的元素用new XX()做参数
```cpp
vector<cls_enemy*> ev;
for (i=0;i<7;i++) {
    ev.push_back(new cls_enemy());
}
```

需要注意vector是模板类型，可以放任何类型，但是基于效率的考虑，最好放对象指针。

