---
layout: post
title: "深度探索C++对象模型：默认构造函数"
date: 2015-03-03 19:36:08 +0800
comments: true
categories: c++ 深度探索C++对象模型
keywords: c++ 默认构造函数 深度探索 c++对象模型
description: 详细介绍了默认构造函数在什么时候构建，以及做了什么事情。
---

>默认构造函数在被需要的时候被编译器产生出来。  

这句话的关键部分是：   

1. 被谁需要？
2. 什么时候被需要？
3. 做什么事情？

### 被谁需要？
编译器为程序构建默认构造函数是因为编译器需要它，而不是因为程序需要它。程序需要什么应该是程序员的责任，应该由程序员提供。举个例子，例如下面这段代码：
{% codeblock lang:cpp %}
class Foo {
public:
  int val;
  Foo* pnext;
}

void foo_bar() {
  Foo bar;
  // 程序需要 bar's members 都被初始化为 0
  if (bar.val || bar.pnext) {
    // do something
  }
}
{% endcodeblock %}    
该段程序的正确语意是希望`Foo`的默认构造函数将`val`和`pnext`初始化为 0，但此处编译器并不会为`Foo`生成默认构造函数，因为这里不满足“需要的时候“这个条件，程序需要不代表编译器需要。后面我们会了解到就算此处编译器为`Foo`生成了默认构造函数，该默认构造函数也不会对`val`和`pnext`进行初始化，具体原因请看“做什么事情？”部分。转载请注明出处：<http://glgjing.github.io/> <!--more--> 
### 什么时候需要？
上面解释了默认构造函数是被编译器需要的，那么编译器什么时候需要它呢？在下列四种情况下编译器会生成默认构造函数：   

1. `class`内包含有`default constructor`的`member object`，合成`default constructor`为了调用`member object`的`default constructor`。
2. `class`继承自含有`default constructor`的基类，合成`default constructor`为了调用基类的`default constructor`。
3. 带有虚函数的`class`，合成`default constructor`主要为了初始化vptr（虚函数表指针）。
4. `class` 有一个及以上的虚基类，合成`default constructor`主要为了初始化虚基类指针。   

当满足上面的条件时，编译器会对`constructor`进行扩展，扩展的规则如下：   

1. 当`class`没有定义`constructor`，编译器会合成`default constructor`，并加入编译器需要的操作，可能包括调用`member object`的`default constructor`，调用基类的`default constructor`，初始化虚函数表指针及虚基类指针。 
2. 当`class`已经定义了一个或多个`constructor`时，编译器不会再去合成`constructor`，但会扩展所有`constructor`加入编译器需要的操作。

### 做什么事情？
从上面的分析中可以看出，默认构造函数所做的事情用一句话可以概括：只执行编译器所需要的行为。所以说即使编译器为`Foo`生成了默认构造函数，该默认构造函数也不会对`val`和`pnext`进行初始化，因为编译器并不需要该行为，这是程序员应该做的事情。
### 总结：
编译器只合成他需要的`default constructor`并且只做他需要的操作。C++ 新手常见的两个<font color="ff0000">误解</font>：
>1. 任何`class`如果没有定义`default constructor`，就会被合成出一个来。
2. 编译器合成出来的`default constructor`会明确设定`class` 内每一个`data member`的默认值。
