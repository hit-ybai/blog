---
layout: post
title: "C++ PRIMER PLUS (10)"
date: 2012-03-10 10:21:31 +0800
comments: true
categories: 读书报告
tags:
- c++
- c++ primer plus
- 读书报告
---
## 第10章 对象和类
### OOP特性：
- 抽象
- 封装和数据隐藏
- 多态
- 继承
- 代码的可重用性

### 10.1 过程性编程和面向对象编程

### 10.2 抽象和类

### 10.3 类的构造函数和析构函数
```cpp
void stock::show() const //promises note change invoking object
```
这种方法声明和定义的类函数成为`const`成员函数，只要类方法不修改调用对象，就应该将其声明为`const`。因为形如`show()`的方法没有形参，不能用`const`引用或`const`指针来避免修改对象。

在当前类的方法中`*this`可作为当前类的别名进行修改或访问。

### 10.7 类的作用域

在类中定义常量的方式——使用关键字`static`:
```cpp
class stock
{
       private:
       static const int len = 30;
       …
};
```
*第十章结束*