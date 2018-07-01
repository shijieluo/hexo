---
title: C++宏
date: 2018-07-01 20:34:20
categories: C++
tags:
---
# 以const，inline，enum 去替换 #define
## 宏与inline函数，const变量及enum的区别
* 宏由预编译器展开，可能未进入符号表；inline由编译器展开，函数名或变量进入符号表。
* 宏没有作用域，一旦定义将在定义后全局范围内作用；inline，const，enum有作用域限制。
* 宏变量不能取地址；const变量可以取地址。

## 在类内使用enum代替初值将在类内被使用的static const 变量
因为有些编译器不支持在类的声明中对static成员变量设置初值。所以使用enum hack的方法可以达到宏的目的，并且这种变量不能被取地址，有自己的作用域。


## Note
- 对于常量，以const对象或enums替换#defines
- 对宏函数，改用inline函数替换#defines


这里需要注意的是，宏函数往往会产生意想不到的错误。比如：

```
#define CALL_WITH_MAX(a,b) f((a) > (b) ? (a) : (b))

int a = 5, b=0;
CALL_WITH_MAX(++a, b);       // a被累加两次  
CALL_WITH_MAX(++a, b+10);    // a被累加一次
```
所以将宏改写成inline函数是有必要的。
