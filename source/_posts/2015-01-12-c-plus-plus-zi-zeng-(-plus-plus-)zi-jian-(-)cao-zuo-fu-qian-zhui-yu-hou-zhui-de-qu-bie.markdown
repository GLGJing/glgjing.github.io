---
layout: post
title: "C++ 自增、自减操作符前缀与后缀的区别"
date: 2015-01-12 20:28:11 +0800
comments: true
categories: C++
keywords: C++ 自增 ++ 自减 -- 前缀 后缀 区别 Increment Decrement 操作符重载
description: 本文介绍了C++自增、自减操作符前缀与后缀的区别。
---
自减和自增操作符规则相同，这里就以自增为例进行说明。C++的自增<code>++</code>操作符分为前缀形式和后缀形式，在语法上前缀与后形式的主要区别在于：前缀形式有时叫做“增加然后取回”，后缀形式叫做“取回然后增加”。例如：
{% codeblock lang:cpp %}
int _tmain(int argc, _TCHAR* argv[])
{
  int a = 0;
  std::cout << a++ << std::endl;
  std::cout << a << std::endl;
  int b = 0;
  std::cout << ++b << std::endl;
  std::cout << b << std::endl;
  return 0;
}
{% endcodeblock %}
{% codeblock lang:cpp %}
0 
1
1
1
{% endcodeblock %}
然而有一个句法上的问题，当对<code>operator ++</code>函数进行重载时，需要自增操作符的前缀和后缀形式有不同的参数。但是不论是自增或自减的前缀还是后缀都只有一个参数。为了解决这个语言问题，<code>C++</code>规定后缀形式有一个<code>int</code>类型参数,当函数被调用时,编译器传递一个<code>0</code>做为<code>int</code>参数的值给该函数，具体实现如下：<!-- more -->
{% codeblock lang:cpp %}
class UPInt {
public:
  UPInt() : value(0) {
  }

  UPInt& operator++() {
    *this += 1;
    return *this;
  }

  const UPInt operator++(int) {
    UPInt old = *this;
    ++(*this);
    return old;
  }

  UPInt& operator+=(int rValue) {
    value += rValue;
    return *this;
  }

  int getValue() {
    return value;
  }

protected:
  int value;
};

int _tmain(int argc, _TCHAR* argv[])
{
  UPInt tmp;
  UPInt a = tmp++; // 调用 const UPInt operator++(int)
  UPInt b = ++tmp; // 调用 UPInt& operator++()
  std::cout << a.getValue() << std::endl;
  std::cout << b.getValue() << std::endl;
  return 0;
}
{% endcodeblock %}
输出结果：
{% codeblock lang:cpp %}
0
2
{% endcodeblock %}
在重载<code>++</code>和<code>--</code>操作符时最好是遵从<code>c++</code>规范，否则会写出让其他人很迷惑的代码。这里有几点需要注意的地方：
<ol>
	<li>前缀形式返回值是引用很好理解，但为什么后缀返回值是<code>const</code>类型？</li>
	<li>看到后缀形式的<code>++</code>实现，是否有效率上的问题？</li>
	<li>为什么后缀形式的<code>++</code>内部没有直接实现自增功能，而是调用了前缀形式的<code>++</code>？</li>
</ol>
<h3>1 为什么后缀返回值是<code>const</code>类型？</h3>
假设不是 const 对象,下面的代码就是正确的:
{% codeblock lang:cpp %}
UPInt i;
i++++; 
{% endcodeblock %}
上面的代码相当于<code>i.operator++(0).operator++(0);</code>。很明显，第一个调用的 <code>operator++</code>函数返回的临时对象调用了第二个<code>operator++</code>函数。这样做的问题是：一是与内置类型行为不一致，当设计一
个类遇到问题时,一个好的准则是使该类的行为与int类型一致，而int类型不允许连续进行两次后缀自增；二是使用两次后缀<code>++</code>所产生的结果与调用者期望的不一致，第二次调用<code>operator++</code>改变的值是第一次调用返回对象的值，而不是原始对象的值。 因此如果<code>i++++;</code>是合法的，<code>i</code>将仅仅增加了一次。
<h3>2 后缀形式的<code>++</code>实现，是否有效率上的问题?</h3>
后缀形式的<code>++</code>函数必须建立一个临时对象以做为它的返回值，上述实现代码建立了一个临时对象(oldValue)，这个临时对象必须被构造并在最后被析构。所以如果仅为了提高代码效率，应该尽量使用前缀形式的<code>++</code>，因为它的效率较高。
<h3>3 为什么后缀形式的<code>++</code>内部没有直接实现自增功能，而是调用了前缀形式的<code>++</code>？</h3>
可以看出后缀与前缀形式的<code>++</code>操作符除了返回值不同外，所完成的功能是一样的。所以需要确保后缀和前缀的<code>++</code>的行为一致性，当不同的程序员去维护和升级代码时，有什么能保证它们不会产生差异？遵守上述代码里的原则，则仅仅需要维护前缀版本,因为后缀形式自动与前缀形式的行为一致。
转载请注明出处：<code>http://glgjing.github.io/</code>。