---
layout: post
title: "深度探索C++对象模型：拷贝构造函数"
date: 2015-03-06 20:47:13 +0800
comments: true
categories: 
---
## 对象之间的拷贝操作
发生对象之间拷贝操作的三种情况：    

1. 用一个 object 对另一个 object 进行初始化操作
2. 当 object 被用作函数参数时
3. 当一个函数返回值为一个 object 时

如果一个`class`明确定义了`copy constructor`，那么在上述几种情况中会调用该`copy constructor`，但如果一个`class`没有定义`copy constructor`呢？转载请注明出处：<http://glgjing.github.io/>。<!--more-->	  

## memberwise Initialization

`memberwise Initialization`的规则是：把每一个内建活继承来的`data member`从一个`object`拷贝到另一个`object`，但并不会拷贝`member class object`，而是以递归的方式对`member class object`施行`memberwise Initialization`。这样的操作是如何完成的呢？   

>从概念上而言，对于一个`class X`，这个操作是被一个`copy constructor`实现出来...     

之所以是“从概念上”，也就是实际上并不是完全由`copy constructor`实现的。   

>一个良好的编译器可以为大部分`class object`产生`bitwise copies`，因为他们有`bitewise copy semantics`...

## bitewise copy semantics（位逐次拷贝）

`bitewise copy semantics`（位逐次拷贝）顾名思义就是把一个`object`的内容按位拷贝给另一个`object`。如果在`bitewise copy semantics`完全可以满足一个`class`的拷贝操作的情况下，编译器是不会合成`copy constructor`。但在有些情况下单纯的`bitewise copy semantics`不能满足拷贝操作，也就是说编译器在一个`class`不会表现出`bitewise copy semantics`时，需要合成`copy constructor`来完成拷贝操作。一个`class`不会表现出`bitewise copy semantics`的四种情况：

1. 当`class`内含有`member object`，而且后者的`class`声明里有一个`copy constructor`，不管是`class`的设计者明确声明的，还是被编译器合成的。
2. 当`class`继承一个`base class`，而后者含有一个`copy constructor`(不管是明确声明的还是编译器合成的)。
3. 当`class`还有`virtual functions`时。
4. 当`class`派生链中有`virtual base classes`时。

## 编译器合成 copy constructor 做什么？  
1. 调用`member object`以及`base class`的`copy constructor`。
2. 重新设定`virtual table`的指针。
3. 处理`virtual base class subobject`

## 总结
>`Default constructor` 和 `copy constructor` 在必要的时候才由编译器产生出来。  

`copy constructor`的合成原则基本上和`default constructor `的合成原则一致，具体可以参考：[
深度探索C++对象模型：默认构造函数 ](http://glgjing.github.io/blog/2015/03/03/shen-du-tan-suo-c-plus-plus-dui-xiang-mo-xing-:mo-ren-gou-zao-han-shu/)。主要原则就是：编译器只合成他需要的`copy constructor`并且只做他需要的操作。