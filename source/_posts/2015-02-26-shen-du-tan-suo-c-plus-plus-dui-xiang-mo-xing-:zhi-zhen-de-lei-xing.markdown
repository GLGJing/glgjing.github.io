---
layout: post
title: "深度探索C++对象模型：指针的类型"
date: 2015-02-26 11:27:18 +0800
comments: true
categories: c++, 深度探索C++对象模型
keywords: c++ 指针 指针类型 深度探索 c++对象模型
description: 介绍了不同类型指针之间的实际区别和联系。
---
指针对于`c++`来说是一个非常强大和灵活的特性，他可以支持多态，可以在不同类型之间进行转换，那么指向不同类型的指针在本质上有什么区别呢？转载请注明出处：<http://glgjing.github.io/>。例如一个类的定义如下：   

{% codeblock lang:cpp %}
class ZooAnimal {
public:
  ZooAnimal();
  virtual ~ZooAnimal();
  virtual void rotate();
protected:
  int loc;
  String name;
};
{% endcodeblock %}
那么下面几个指针有什么不同：   

{% codeblock lang:cpp %}
ZooAnimal* px;
int* pi;
Array<String>* pta;
{% endcodeblock %}
单纯从内存需求上来看，没有什么不同，在 32 位机器上都是 4 个字节，也就是一个机器地址大小。从存储的内容来看，也没什么不同，都是存储一个地址。真正不同的是寻址出来的内容类型不同，也就是说：   
>"指针类型"会教导编译器如何解释某个特定地址中的内存内容及其大小。   

所以说虽然一个`void*`指针可以指向一个地址，但是我们无法通过`void*`来操作该地址所存储的内容，因为编译器无法知道应该用什么方式来解释和操作该内容，这时需要进行类型转换（`cast`）来告诉编译器如何进行操作。    
>所以，转型(`cast`)其实是一种编译器指令。大部分情况下它并不改变一个指针所含的真正地址，它只影响“被指出之内存的大小和其内容”的解释方式。



