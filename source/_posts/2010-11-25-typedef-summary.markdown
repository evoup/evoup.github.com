---
layout: post
title: "typedef小结"
date: 2010-11-25 14:52
comments: true
categories:           c-language
---
在头文件里typedef的再inlcude到源文件里，这样就可以省去struct xx obj;直接xx obj;

对于动态分配内存的时候也可以

```c++
xx * obj= (xx *)malloc(sizeof(xx));
```

