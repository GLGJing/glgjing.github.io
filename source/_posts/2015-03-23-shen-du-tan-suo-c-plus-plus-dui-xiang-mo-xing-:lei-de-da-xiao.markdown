---
layout: post
title: "深度探索C++对象模型：类的大小"
date: 2015-03-23 19:21:06 +0800
comments: true
categories: c++ 深度探索C++对象模型
keywords: c++ 对象模型 深度探索 c++对象模型 类的大小 sizeof
---
现在有这样一段代码：
{% codeblock lang:cpp %}
class X {
};

class Y : public virtual X {
};

class Z : public virtual X {
};

class A : public Y, public Z {
};
{% endcodeblock %}
对 `class X、Y、Z、A` 进行 `sizeof` 运算结果是什么呢？如果你心里已经有了答案，不妨看完下面的分析再说。转载请注明出处：<http://glgjing.github.io/>    

`C++` 初学者常有一个错误的认识，认为一个 `class` 的大小就是该 `class` 内所有 `non static members` 的大小总和。既然是错误的认识，那么实际上就不是或者不单纯是该 `class` 内所有 `non static members` 的大小总和。在 `c++` 对象模型里，一个 `class` 的大小主要受下面三个方面影响：<!--more-->

### 1 语言本身特性造成的额外负担
这里的额外负担主要是受 `virtual` 特性影响，包括 `virtual base class` 和 `virtual function`。在 `derived class` 中这种额外负担反映在某种形式的指针上，可以是虚函数指针，或者指向虚基类的 `subject class` 偏移地址等。详细内容可以参考[C++ 虚函数浅析](http://glgjing.github.io/blog/2015/01/03/c-plus-plus-xu-han-shu-qian-xi/)。

### 2 编译器对特殊情况的优化处理
例如对于 `empty virtual base class` 的特殊支持，在 `C++` 对象模型中一个空的 `class` 大小为 `1 byte`，因为编译器为了使每个 `object` 在内存中能有独一无二的地址，为空 `class` 安插了一个 `char`。如果一个空的 `class` 大小为 `0 byte`，那么像该 `class` 的数组 `X x[10];`，编译器将无法区分每个元素的地址。但是如果 `derived class` 的大小不为 `0`，就不需要 `base class` 的 `1 byte` 进行内存地址区分，那么如果编译器对这种情况进行了优化的话，就会将该 `1 byte` 去掉，如果没有还是会继续保留这个额外的 `1 byte` 空间。  

说到编译器对特殊情况的优化，下面这段话比较恰当表述了编译器演化与`C++`对象模型的关系：
>编译器之间的潜在差异正说明了 `C++` 对象模型的演化。这个模型为一般情况提供了解决之道，当特殊情况渐渐被挖掘出来时，种种启发法于是被引入，提供优化的处理。如果成功，启发法于是就提升为普遍的策略，并跨越各种编译器而合并。他被视为标准（虽然他并不被规范为标准），久而久之也就成了语言的一部分。

### 3 边界调整（Alignment）的影响
为了数据能够更有效率的在内存中存取，编译器会 `class` 的内存边界进行调整，在`32`位机器上通常 `alignment` 为 `4 bytes`，以使总线的运输效率最高。

经过上面的分析后，可以看出具体结果要视编译器而定。首先讨论在编译器没有对 `empty virtual base class` 进行优化的情况，那么：

1. `class X`  的大小应该是 `1 byte`，该 `1 byte` 由编译器插入。
2. `class Y` 和 `class Z` 的大小相同为 `8 `，包括 `4 byte` 的 虚基类指针再加上 `1 byte` 的基类 X的大小，考虑到内存对齐需要补齐 `3 byte`，所以最终结果为 `8 byte`。
3. `class A` 的大小为`12 byte`，包括 `class Y` 和 `class Z` 内的两个 `4 byte` 虚基类指针共 `8 byte`，此外由于 `X` 为虚基类，所以在 `A` 内只有一个实体，所以应该再加上 `X` 的 `1 byte` 和内存对齐额外补齐的 `3 byte`，共 `12 byte`。   
 
所以最终结果为：
{% codeblock lang:cpp %}
sizeof(X) = 1 byte
sizeof(Y) = sizeof(Z) = 8 byte
sizeof(A) = 12 byte。
{% endcodeblock %}
如果编译器对 `empty virtual base class` 进行了优化，那么 `class X` 的 `1 byte` 空间在派生类中将被拿掉，相应的由于内存对齐而引起的 `3 byte` 也被去掉了，所以最终结果应该是：
{% codeblock lang:cpp %}
sizeof(X) = 1 byte
sizeof(Y) = sizeof(Z) = 4 byte
sizeof(A) = 8 byte。
{% endcodeblock %}