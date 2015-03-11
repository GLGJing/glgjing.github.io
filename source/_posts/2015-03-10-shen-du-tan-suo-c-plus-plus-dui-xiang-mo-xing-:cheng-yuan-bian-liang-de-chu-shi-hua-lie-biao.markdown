---
layout: post
title: "深度探索C++对象模型：成员变量的初始化列表"
date: 2015-03-10 20:21:00 +0800
comments: true
categories: 
---
对`class members`的初始化操作可以放在`member initialization list`或者`construct`中进行，但两者之间是有一些区别的，无论是在效率上还是在语法上。    
## 何时应该使用 initialization list ？
在语法方面来说，为了程序可以通过编译，下面几种情况必须使用 `initialization list`：  

1. 当初始化一个`reference member`时
2. 当初始化一个`const member`时
3. 当调用一个`base class`的`constructor`，并且它拥有一组参数时
4. 当调用一个`member object`的`constructor`，并且它拥有一组参数时

至于为什么编译器会有如此要求，看了后面的编译器扩展内容应该就会明白了。转载请注明出处：<http://glgjing.github.io/><!--more-->    

下面来看一下在效率方面什么时候应该用`initialization list`。首先看下面一段代码：
{% codeblock lang:cpp %}
class Word {
public:
  Word() {
    _name = 0;
    _cnt = 0;
  }
protected:
  String _name;
  int _cnt;
}
{% endcodeblock %}
上面这段代码没有语法错误，程序可以正确的编译执行，但是效率上却不尽人意。首先我们来看编译都干了什么，下面是经过编译器扩展后的`constructor`伪码：
{% codeblock lang:cpp %}
// C++ 伪码
Word::Word {
  // 调用 _name 的默认构造函数
  _name.String::String();
  // 生成临时对象
  String temp = String(0);
  // 将临时对象 memberwise 拷贝给 _name
  _name.String::Operator=(temp);
  // 销毁临时对象
  temp.String::~String();
  _cnt = 0;
}
{% endcodeblock %}
通过编译器扩展后的代码我们可以看到，这种方式的`_name`初始化效率很差，需要产生一个临时对象，多了一次构造、一次拷贝、一次析构的额外操作。不过通过这种初始化方式对`_cnt`并没有什么影响。所以改进后的代码应该是这样的：
{% codeblock lang:cpp %}
Word::Word() : _name(0) {
  _cnt = 0;
}
{% endcodeblock %}
经过编译器扩展后的伪码类似这样：
{% codeblock lang:cpp %}
Word::Word() {
  _name.String::String(0);
  _cnt = 0;
}
{% endcodeblock %}
## member initialization list 到底都做了什么？
通过上面的分析，可以大概看出`initialization list`的作用，具体细节是：   
>编译器会一一操作`initialization list`，以适当的次序在`constructor`之内安插初始化操作，并且在任何`explicit user code`之前。

## member initialization list 存在的一些风险
上面所说的**适当次序**是指`members`在`class`里的声明次序，而不是在`initialization list`的排列次序，所以例如下面这段代码就会问题：
{% codeblock lang:cpp %}
Class X {
  int i;
  int j;
 
public:
  X(int val) : j(val), i(j) {
  }
}
{% endcodeblock %}
上面代码的本意是先用`val`初始化`j`在用`j`初始化`i`，但实际上`initialization list`的初始化次序是按照`members`在`class`里的声明次序，所以会先初始化`i`，然后才是`j`，而`j`一开始并未进行初始化，导致`i(j)`的结果也无法预料。
