---
layout: post
title: "C语言宏定义技巧(常用宏定义)(转)"
date: 2011-01-16 13:43
comments: true
categories:      c-language
---
写好C语言，漂亮的宏定义很重要，使用宏定义可以防止出错，提高可移植性，可读性，方便性 等等。下面列举一些成熟软件中常用得宏定义。。。。。。 

1，防止一个头文件被重复包含 
```c
#ifndef COMDEF_H 

#define COMDEF_H 

//头文件内容 

#endif 
```

<!-- more -->


2，重新定义一些类型，防止由于各种平台和编译器的不同，而产生的类型字节数差异，方便移植。 
```c
typedef unsigned char boolean; /* Boolean value type. */ 
typedef unsigned long int uint32; /* Unsigned 32 bit value */ 
typedef unsigned short uint16; /* Unsigned 16 bit value */ 
typedef unsigned char uint8; /* Unsigned 8 bit value */ 
typedef signed long int int32; /* Signed 32 bit value */ 
typedef signed short int16; /* Signed 16 bit value */ 
typedef signed char int8; /* Signed 8 bit value */ 
```

//下面的不建议使用
```c 
typedef unsigned char byte; /* Unsigned 8 bit value type. */ 
typedef unsigned short word; /* Unsinged 16 bit value type. */ 
typedef unsigned long dword; /* Unsigned 32 bit value type. */ 
typedef unsigned char uint1; /* Unsigned 8 bit value type. */ 
typedef unsigned short uint2; /* Unsigned 16 bit value type. */ 
typedef unsigned long uint4; /* Unsigned 32 bit value type. */ 
typedef signed char int1; /* Signed 8 bit value type. */ 
typedef signed short int2; /* Signed 16 bit value type. */ 
typedef long int int4; /* Signed 32 bit value type. */ 
typedef signed long sint31; /* Signed 32 bit value */ 
typedef signed short sint15; /* Signed 16 bit value */ 
typedef signed char sint7; /* Signed 8 bit value */ 
```


3，得到指定地址上的一个字节或字 
```c
#define MEM_B( x ) ( *( (byte *) (x) ) ) 
#define MEM_W( x ) ( *( (word *) (x) ) ) 
```

4，求最大值和最小值 
```c
#define MAX( x, y ) ( ((x) > (y)) ? (x) : (y) ) 
#define MIN( x, y ) ( ((x) < (y)) ? (x) : (y) ) 
```

5，得到一个field在结构体(struct)中的偏移量 
```c
#define FPOS( type, field ) \ 

/*lint -e545 */ ( (dword) &(( type *) 0)-> field ) /*lint +e545 */ 
```

6,得到一个结构体中field所占用的字节数 
```c
#define FSIZ( type, field ) sizeof( ((type *) 0)->field ) 
```

7，按照LSB格式把两个字节转化为一个Word 
```c
#define FLIPW( ray ) ( (((word) (ray)[0]) * 256) + (ray)[1] ) 
```

8，按照LSB格式把一个Word转化为两个字节 
```c
#define FLOPW( ray, val ) \ 

(ray)[0] = ((val) / 256); \ 

(ray)[1] = ((val) & 0xFF) 
```

9，得到一个变量的地址（word宽度） 
```c
#define B_PTR( var ) ( (byte *) (void *) &(var) ) 
#define W_PTR( var ) ( (word *) (void *) &(var) ) 
```

10，得到一个字的高位和低位字节 
```c
#define WORD_LO(xxx) ((byte) ((word)(xxx) & 255)) 
#define WORD_HI(xxx) ((byte) ((word)(xxx) >> 8)) 
```

11，返回一个比X大的最接近的8的倍数 
```c
#define RND8( x ) ((((x) + 7) / 8 ) * 8 ) 
```

12，将一个字母转换为大写 
```c
#define UPCASE( c ) ( ((c) >= 'a' && (c) <= 'z') ? ((c) - 0x20) : (c) ) 
```

13，判断字符是不是10进值的数字 
```c
#define DECCHK( c ) ((c) >= '0' && (c) <= '9') 
```

14，判断字符是不是16进值的数字 
```c
#define HEXCHK( c ) ( ((c) >= '0' && (c) <= '9') ||\ 
((c) >= 'A' && (c) <= 'F') ||\ 
((c) >= 'a' && (c) <= 'f') ) 
```

15，防止溢出的一个方法 
```c
#define INC_SAT( val ) (val = ((val)+1 > (val)) ? (val)+1 : (val)) 
```

16，返回数组元素的个数 
```c
#define ARR_SIZE( a ) ( sizeof( (a) ) / sizeof( (a[0]) ) )
``` 

17，返回一个无符号数n尾的值MOD_BY_POWER_OF_TWO(X,n)=X%(2^n) 
```c
#define MOD_BY_POWER_OF_TWO( val, mod_by ) \ 
( (dword)(val) & (dword)((mod_by)-1) ) 
```

18，对于IO空间映射在存储空间的结构，输入输出处理 
```c
#define inp(port) (*((volatile byte *) (port))) 
#define inpw(port) (*((volatile word *) (port))) 
#define inpdw(port) (*((volatile dword *)(port))) 
#define outp(port, val) (*((volatile byte *) (port)) = ((byte) (val))) 
#define outpw(port, val) (*((volatile word *) (port)) = ((word) (val))) 
#define outpdw(port, val) (*((volatile dword *) (port)) = ((dword) (val))) 
```

[2005-9-9添加] 

19,使用一些宏跟踪调试 

A N S I标准说明了五个预定义的宏名。它们是： 

_ L I N E _ 

_ F I L E _ 

_ D A T E _ 

_ T I M E _ 

_ S T D C _ 

如果编译不是标准的，则可能仅支持以上宏名中的几个，或根本不支持。记住编译程序 

也许还提供其它预定义的宏名。 

_ L I N E _及_ F I L E _宏指令在有关# l i n e的部分中已讨论，这里讨论其余的宏名。 

_ D AT E _宏指令含有形式为月/日/年的串，表示源文件被翻译到代码时的日期。 

源代码翻译到目标代码的时间作为串包含在_ T I M E _中。串形式为时：分：秒。 

如果实现是标准的，则宏_ S T D C _含有十进制常量1。如果它含有任何其它数，则实现是 

非标准的。 

可以定义宏，例如: 

当定义了_DEBUG，输出数据信息和所在文件所在行 
```c
#ifdef _DEBUG 
#define DEBUGMSG(msg,date) printf(msg);printf(“%d%d%d”,date,_LINE_,_FILE_) 
#else 
#define DEBUGMSG(msg,date) 
#endif 
```

20，宏定义防止使用是错误 

用小括号包含。 

例如：#define ADD(a,b) （a+b） 

用do{}while(0)语句包含多语句防止错误 

例如：#difne DO(a,b) a+b;\ 

a++; 

应用时：if(….) 

DO(a,b); //产生错误 

else 


解决方法: #difne DO(a,b) do{a+b;\ 

a++;}while(0)