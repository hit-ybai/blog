---
layout: post
title: "C++ PRIMER PLUS (7)"
date: 2012-01-29 08:21:31 +0800
comments: true
categories: 读书报告
tags:
- c++
- c++ primer plus
- 读书报告
---
##第七章 函数——C++编程模块 
####7.1.1 定义函数 
如果声明的返回类型是`double`，而函数返回一个`int`表达式，则该`int`值将被强制转换为`double`类型 


####7.1.2 函数原型和函数调用 
句法：函数原型不要求提供变量名，有类型列表就足够了，如：
```cpp
void test(int);
```
原型用的变量名相当于占位符，因此不必与函数定义中的变量名相同
功能：原型确保以下几点：
- 编译器正确处理函数的返回值
- 编译器检查使用的参数数目是否正确
- 编译器检查使用的参数数目是够正确

通常，原型自动将被传递的参数强制转换为期望的类型。当较大的类型被自动转换为较小的类型时，有些编译器将发出警告，指出这可能会丢失数据 

###7.2 函数参数和按值传递 

###7.3 函数和数组 
在C++中，当且仅当用于函数头或函数原型中时，`int *arr`和`int arr[]`的含义才是相同的。
当指针指向数组的第一个元素时，本书中使用数组表示法；而当指针指向一个独立的值时，使用指针表示法。
```cpp
arr[i] == *(ar + i);
&arr[i] == ar+(i * 指向类型的长度)
```
原数组和参数数组指向同一个地址。但`sizeof(原数组)`为数组大小，而`sizeof(参数数组)`则为指针变量的大小，这就是必须显示传递数组长度，而不能再函数中使用`sizeof()`计算的原因；
记住：为将数组类型和元素属相告诉数组处理函数，请通过两个不同的参数来传递它们。
```cpp
void fillArray(int arr[], int size); //prototype
void fillArray(int arr[size]); //bad prototype 
```

####7.3.5 指针和const 
```cpp
int age = 39;
const int *pt  = &age;
```
该声明指出，`pt`指向一个`const int(这里是39)`，因此不能使用pt来修改这个值，换句话来说，`*pt`的值为`const`，不能被修改：
```cpp
*pt += 1; //INVALID
cin >> *pt; //INVALID    
```
尽可能使用`const`，将指针参数声明为指向常量数据的指针有两条理由
1. 这样可以避免无意间修改数据而导致变成错误;
2. 使用`const`使得函数能够处理`const`和非`const`实参，否则只能接受非`const`数据。

第二种使用`const`的方式使得无法修改指针的值：
```cpp
       int sloth = 3;
       int * const finger = &sloth;
```
这种声明格式使得`finger`只能指向`sloth`，但允许使用`finger`来修改`sloth`的值。 

###7.4 函数和二维数组
```cpp
int sum(int (*arr)[常数], int size);
int sum(int arr[][常数], int size);
```
处理固定行数，未知列数的二维数据；
```cpp
ar2[r][c] == *(*(ar2 + r) + c); 
```

###7.9 函数指针 
与数据项相似，函数也有地址。函数地址是存储机器语言代码的内存的开始地址。 

####7.9.1 函数指针的基础知识 
1. 获取函数的地址
只要使用函数名即可。也就是说，如果`think()`是一个函数，则`think`就是该函数的地址。

2. 声明函数指针
```cpp
double pam(int);
double (*pf)(int);
```
    使用该函数指针的函数的原型
```cpp
void estimate(int lines, double(*pf)(int));
```
3. 使用指针来调用函数
```cpp
double x = (*pf)(5); 
```

注：C++中`(*pf)(5)`和`pf(5)`是等价的。 

*第七章结束*