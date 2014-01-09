---
layout: post
title: "freebsd上手动编译boost的boost-regex库"
date: 2014-01-09 14:45
comments: true
categories: [cplusplus]
---

Boost这玩意用起来容易，安装起来却很麻烦，如果按照默认安装很庞大臃肿，原因是bjam搞得太复杂了。对于只使用其中某几个模块而采用整体安装的方式时间和空间上都是划不来的。如果只需要连接其中的正则表达式模块regex的话，可以采取单独安装的方法。

<!-- more -->

###1.生成bjam编译器
boost不同的版本可能会有bjam包的路径不同的情况，这里采用boost_1_38_0为例。
```sh
$ pwd
/home/software
$ fetch http://sourceforge.net/projects/boost/files/boost/1.38.0/boost_1_38_0.tar.bz2 
$ tar xjf boost_1_38_0.tar.bz2
$ cd tools/jam/src
$ ./build.sh
```

编译后bjam程序存放于tools/jam/src/bin.freebsdx86_64/
```sh
vi ./tools/build/v2/user-config.jam
```
修改“#using gcc;”为"using gcc;"

###2.用bjam编译boost
```sh
$ ./bjam clean #清理之前的安装
$ ./bjam --build-type=complete --with-regex workdir #只编译regex库，生成include和lib
```
注意bjam编译时，需要注意bjam只是把boost的include和lib的路径放在本地，一个是当前目录的子目录boost中，一个是在刚在指定workdir的lib中。
然后复制到freebsd的include目录(/usr/local/include)和lib目录(/usr/local/lib)。
```sh
cd /home/software/boost_1_38_0
sudo cp -r boost /usr/local/include/
sudo cp -r workdir/lib /usr/local/
```

###3.编译使用boost-regex的程序
简单的程序regex.cpp试一把
```cpp
#include <boost/regex.hpp>
#include <iostream>

int main() {
 	std::string s = "<html>我是boost正则表达式匹配到的内容</html>";
 	boost::regex reg( "<html>(.+?)</html>" );
 	boost::sregex_token_iterator p( s.begin(), s.end(), reg, 01 );
 	boost::sregex_token_iterator end;
 	while ( p != end ) {
     	std::cout << *p++ << std::endl;
 	}
 	return 1;
}
```
-----------付测试方法-----------
###编译方法
```sh
$ g++ -Wall regex.cpp -lboost_regex-gcc42-d-1_38 -I/usr/local/include -L/usr/local/lib -o regex
$ ./regex
我是boost正则表达式匹配到的内容
```

这样就妥妥的了，祝各位看官好运，收工:)