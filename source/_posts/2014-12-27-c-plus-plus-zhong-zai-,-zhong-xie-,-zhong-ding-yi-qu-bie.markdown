---
layout: post
title: "c++重载、重写、重定义区别"
date: 2014-12-27 15:32:17 +0800
comments: true
categories: C++
keywords: C++ 重载 重定义 重写 覆盖
description: 简单总结一下，c++重载、重写、重定义区别。
---
<h2>一 重载（overload）</h2>
<h4>概念：</h4>
函数有同样的名称，但是参数列表不相同的情形，这样的同名不同参数的函数之间，互相称之为重载函数。
<h4>基本条件：</h4>
<ul>
	<li>函数名必须相同；</li>
	<li>函数参数必须不相同，可以是参数类型或者参数个数不同；</li>
	<li>函数返回值可以相同，也可以不相同；</li>
</ul>><!-- more -->
<h4>注意：</h4>
<ul>
	<li>只能通过不同的参数样式进行重载，例如：不同的参数类型，不同的参数个数，不同的参数顺序；</li>
	<li>不能通过访问权限、返回类型、抛出的异常进行重载；</li>
	<li>重载的函数应该在相同的作用域下；</li>
</ul>
<h4>验证程序：</h4>
{% codeblock lang:cpp %}
class A {
 public:
  void Func1(int arg1) {
    std::cout << "func 1" << std::endl;
  }

  // OK: 通过参数类型不同重载 Func1()
  void Func1(double arg1) {
    std::cout << "func 2" << std::endl;
  }

  // OK: 通过参数个数不同重载 Func1()
  void Func1(int arg1, int arg2) {
    std::cout << "func 3" << std::endl;
  }

  // OK: 重载函数返回值可以不同
  bool Func1(int arg1, double arg2) {
    std::cout << "func 4" << std::endl;
    return true;
  }

  // ERROR: 不能只通过返回值来进行重载
  /*bool Func1(int arg1) {
    return true;
  } */
};

int _tmain(int argc, _TCHAR* argv[])
{
  A a;
  a.Func1(1);
  a.Func1(1.0);
  a.Func1(1, 2);
  a.Func1(1, 2.0);
  return 0;
}

{% endcodeblock %}
输出：
{% codeblock %}
func 1
func 2
func 3
func 4
{% endcodeblock %}
<h2>二 重写（override）</h2>
<h4>概念：</h4>也称为覆盖，子类重新定义父类中有相同名称和参数的虚函数，主要在继承关系中出现。
<h4>基本条件：</h4>
<ul>
	<li>重写的函数和被重写的函数必须为virtual函数，分别位于基类和派生类中；</li>
	<li>重写的函数和被重写的函数函数名和函数参数必须一致；</li>
	<li>重写的函数和被重写的函数返回值相同，或者都返回指针或引用，并且派生类虚函数所返回的指针或引用的类型是基类中被替换的虚函数所返回的</li>
	<li>指针或引用的类型的子类型。</li>
</ul>
<h4>注意：</h4>
<ul>
	<li>重写的函数所抛出的异常必须和被重写的函数所抛出的异常一致，或者是其子类；</li>
	<li>重写的函数的访问修饰符可以不同于被重写的函数，如基类的virtual函数的修饰符为private，派生类改为public或protected也是可以的。</li>
	<li>静态方法不能被重写，也就是static和virtual不能同时使用。</li>
	<li>重写的函数可以带virtual关键字，也可以不带。</li>
</ul>
<h4>验证程序：</h4>
{% codeblock lang:cpp %}
class A {
};

class B : public A {
};

class C {
 public:
  virtual void Func1() {
    std::cout << "class C: func 1" << std::endl;
  }

  virtual A* Func2() {
    std::cout << "class C: func 2" << std::endl;
    return new A;
  }

  // ERROR:静态函数不能被声明为virtual，也就没办法被重写。
  // static virtual void FuncStatic() {}

  //由于Func3被声明为private，所以需要通过public函数来调用
  void ShowFunc3() {
    Func3();
  }

  virtual void Func4() {
    std::cout << "class C: func 4" << std::endl;
  }

 private:
  virtual void Func3() {
    std::cout << "class C: func 3" << std::endl;
  }
};

class D : public C {
 public:
  // OK: 重写C类Func1，可以不带virtual关键字
  void Func1() {
    std::cout << "class D: func 1" << std::endl;
  }

  // OK: 当返回值为指针或者引用时，返回值可以是父类返回值类型的子类
  virtual B* Func2() {
    std::cout << "class D: func 2" << std::endl;
    return new B;
  }

  // ERROR: 除上面的情况，返回值类型要和父类一直
  /*virtual bool Func2() {
  }*/

  // OK: 重写的函数的访问修饰符可以不同于被重写的函数
  virtual void Func3() {
    std::cout << "class D: func 3" << std::endl;
  }

 private:
  // OK: 重写的函数的访问修饰符可以不同于被重写的函数
  virtual void Func4() {
    std::cout << "class D: func 4" << std::endl;
  }
};

int _tmain(int argc, _TCHAR* argv[])
{
  C* c = new D;
  c->Func1();
  c->Func2();
  c->ShowFunc3();
  c->Func4();
  return 0;
}

{% endcodeblock %}
输出：
{% codeblock %}
class D: func 1
class D: func 2
class D: func 3
class D: func 4
{% endcodeblock %}
<h2>三 重定义（redefining）</h2>
<h4>概念：</h4>也叫隐藏，子类重新定义父类中的非虚函数，屏蔽了父类的同名函数
<h4>基本条件：</h4>
<ul>
	<li>被隐藏的函数之间作用域不相同</li>
</ul>
<h4>注意：</h4>
<ul>
	<li>子类和父类的函数名称相同，但参数不同，此时不管父类函数是不是virtual函数，都将被隐藏。</li>
	<li>子类和父类的函数名称相同，参数也相同，但是父类函数不是virtual函数，父类的函数将被隐藏。</li>
</ul>
<h4>验证程序：</h4>
{% codeblock lang:cpp %}
class A {
public:
	void Func1(int a) {
		std::cout << "class A: Func1" << std::endl;
	}

	virtual void Func2(int a) {
		std::cout << "class A: Func2" << std::endl;
	}
};

class B : public A {
public:
	// 将会重定义父类的方法
	void Func1(int a) {
		std::cout << "class B: Func1" << std::endl;
  }

  // 将会重写(override)父类方法
  void Func2(int a) {
    std::cout << "class B: Func2-1" << std::endl;
  }

  // 将会重定义父类的方法
  void Func2(int a, int b) {
    std::cout << "class B: Func2-2" << std::endl;
  }
};

int _tmain(int argc, _TCHAR* argv[])
{
  B* b = new B;
  b->Func1(1);
  b->Func2(1);
  b->Func2(1, 1);
  return 0;
}

{% endcodeblock %}
输出：
{% codeblock %}
class B: Func1
class B: Func2-1
class B: Func2-2
{% endcodeblock %}
<h2>四 总结</h2>
重载、重写、重定义书面上的区别，以及各自的规则没有太大意义，而且这些名词本身都是翻译过来的，不同的地方翻译也不尽相同，如果初学C++，弄清每个概念的实际意义，以及为什么这么设计才是最重要的。转载请注明出处：<code>http://glgjing.github.io/</code>。
