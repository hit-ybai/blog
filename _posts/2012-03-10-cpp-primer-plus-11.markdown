---
layout: post
title: "C++ PRIMER PLUS (11)"
date: 2012-03-10 15:21:31 +0800
comments: true
categories: 读书报告
tags:
- c++
- c++ primer plus
- 读书报告
---
##第11章 使用类
###11.1 操作符重载
操作符重载时一种形式的C++多态，函数原型：
```cpp
Type_name operator op (argument-list);
```
`Type_name operator op`相当于函数名，`(argument-list)`相当于参数列表，`Type_name`相当于返回值，只是在调用的时候可省去函数后的括号和函数名中`operator`（不省也行，如`strsum = str1.operator+(str2);`）

第一个参数通过`this`指针隐式传递
###11.3 友元简介
在左侧的操作数不是调用对象时，可使用非成员函数
```cpp
       A = 2.75 * B; // can not correspond to a mumber fuction
       A = operator*(2.75, B); // can correspond to a mumber fuction
```
函数原型如下：
```cpp
       Time operator* (double m, const Time &t);
```
但这引发了一个新问题，非成员函数不能访问类的私有数据。
创建友元函数要先将其原型放在类声明中，并在前面加上关键字`friend`：
```cpp
       friend Time operator* (double m, const Time &t);
```
友元函数不能使用成员操作符调用，但与成员函数访问权限相同
```cpp
       Time operator* (double m, const Time &t)
       {
              return t * m;
       }
```
重载`<<`操作符

第一种版本，定义友元函数：
```cpp
       void operator<< (ostream &os, const Time &t)
       {
              os << t.hours << "hours, " << t.minutes << "minutes";
       }
```
`os`可以暂时理解为`cout`的别名
###11.5 再谈重载：矢量类
```cpp
              shove.set(100, 300); //直接设置
              shove = Vector(100, 300) //Vector为类名，此方法使用构造函数创建一个临时对象，让后赋给shove
```
一元负号操作符
```cpp
              Vector Vector::operator-() const
              {
                     return Vector(-x, -y);
              }
```
###11.6 类的自动转换和强制类型转换
只接受一个参数的构造函数如：
```cpp
       Stonewt (double lbs);
```
可隐式转换
```cpp
       Stonewt mycat;
       mycat = 19.6; //程序调用Stonewt(double)来创建临时对象，然后赋值
       //但这种类型并不总合乎需要，可用explicit来关闭这种特性
      explicit Stonewt(double lbs);
```
这将关闭上例的隐式转换，但仍然允许显式强制转换
```cpp
       Stonewt myCat；
       myCat = Stonewt(19.6);
       myCat = (Stonewt) 19.6;
```
现在研究如何将对象转换为数字
转换函数：
```cpp
       operator typeName();
```
注意：
- 转换函数必须是类方法
- 转换函数不能指定返回类型
- 转换函数不能有参数

例如，转换为double的函数原型
```cpp
       operator double();
```
如果该类值定义了一个转换函数，则可以直接用`cout`输出该类(`<<`未重载。否则编译器将认为语句有二义性而拒绝它，只能在前增加强制类型转换
转换函数也有其优缺点，如：
```cpp
       int ar[20];
       ……
       Stonewt temp(14, 4);
       int Temp = 1;
       cout << ar[temp];
```
这种误用对象为索引之类的错误不会被编译器发现，而转换函数又不能用`explicit`来关闭隐式转换
解决方法是定义一个功能相同的非转换函数
```cpp
       int Stonest::Stone_to_Int()
       {
              return int(pounds + 0,5);
       }
```
在`main`上面定义的全局对象的构造函数会先于`main`运行

*第十一章结束*