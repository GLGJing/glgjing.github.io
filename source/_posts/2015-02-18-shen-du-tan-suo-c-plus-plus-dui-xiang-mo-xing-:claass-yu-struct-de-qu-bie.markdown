---
layout: post
title: "深度探索C++对象模型：class 与 struct 的区别"
date: 2015-02-18 01:32:20 +0800
comments: true
categories: c++ 深度探索C++对象模型
keywords: c++ class struct 区别 深度探索 c++对象模型
description: 介绍了 class 和 struct 的区别。
---
`class`和`struct`的区别到底是什么，什么时候应该用`struct`取代`class`呢？我想大多数从`C`转到`C++`的程序员都会有过这样的困惑。看了`Lippman`的《深度探索C++对象模型》对该问题的回答之后，我只想说一句：尼玛，只要跟哲学沾边的问题就注定不会有一个简单明了的答案！    

首先引用`Lippman`的一句话：   
>嘿，你知道吗，struct 那个关键字其实没什么用。。。“什么时候应该用 struct 取代 class ？” 答案之一是：当他让一个感觉比较好的时候。   

大部分`C++`程序员都知道，如果单纯从语法角度来区分`struct`和`class`的话，其实除了以下几点没有什么不同：    

1. 默认继承权限不同。`class`的默认继承权限为`private`，而`struct`的默认继承权限为`public`；
2. 成员的默认访问权限不同。`class`的成员默认是`private`权限，`struct`默认是`public`权限。
3. `struct`不能作为模板参数，而`class`可以。 

那'Lippman'为什么不直说呢？偏得之乎者也的提升到精神层次，搞的读者一头雾水。<!--more-->个人觉得主要是语法上的区别只是结果，而为什么这么设计才是`Lippman`想传达的主要内容。在`C`所支持的`struct`和`C++`所支持的`class`之间有一个观念上的重要差异，而关键词本身并不提供这种差异。你可能会问如果说为了兼容`C`而提供了`struct`关键字，那干嘛还创建`class`这个关键字，都用`struct`不行吗？当然行，但是`Lippman`觉得`class`更能令人满意（其实就是他自己看着更爽），更能表达出封装和继承的哲学。   

所以如果你还在电脑前纠结，请释怀吧！谁让人家是`C++`形象代言人呢，就是这么任性，就像当面对`struct`用在`template`参数时无法被语意分析器`parser`处理时，你只能迷茫的问为什么，而`Lippman`可以默默的把`parser`改掉。最后你可以说`struct`的保留是为了兼容`C`；也可以说`class`的引入更具哲学意义；还可以说他们的共存是为了更多程序员从`C`迁移到`C++`部落。转载请注明出处：<http://glgjing.github.io/>。