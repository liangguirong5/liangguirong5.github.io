---
title: C++之初始化列表
tags:
  - C++
categories: C++
date: 2017-02-03 15:31:41
---

# 什么是初始化列表

初始化列表是构造函数的一个组成部分，可以有可以没有，因此构造函数包括：函数体、参数列表、函数名以及初始化列表。

<!-- more -->

```C++
struct foo
  {
    string name;
  	int id;
  	foo(string s, int i):name(s),id(i){};
  };
class C: public B2, public B1, public B3
{
public:
    C(int a, int b, int c, int d):B1(a), memberB2(d), memberB1(c),B2(b){}
private:
    B1 memberB1;
    B2 memberB2;
    B3 memberB3;
};
```

# 初始化列表的作用

初始化列表的作用是对成员变量进行赋值，当然也可以在函数体里面进行赋值操作，但是对于class类型的成员变量，用初始化列表来赋值可以减少一次默认构造的过程，从而提高性能。下面两段代码解释为什么可以减少默认构造。

```c++
struct Test2
{
    Test1 test1 ;
    Test2(Test1 &t1)
    {
        test1 = t1 ;
    }
};
```

```c++
struct Test2
{
    Test1 test1 ;
    Test2(Test1 &t1):test1(t1){}
}
```

分别执行如下代码

```c++
Test1 t1 ;
Test2 t2(t1) ;
```

对于第一段代码，要先初始化test1，也就是先调用类Test1的默认构造函数，然后调用赋值构造函数（等号重载）；对于第二段代码，只需要直接调用类Test1的拷贝构造函数，对test2进行初始化，因此初始化列表可以减少一次默认初始化。

# 必须使用初始化列表的情况

同时，根据上面的例子我们可以知道，当Test1类没有默认构造函数时，我们必须采用初始化列表的方式。

# 阅读更多

[C++初始化列表](http://www.cnblogs.com/graphics/archive/2010/07/04/1770900.html)

[虚基类实现机制](http://blog.csdn.net/jiangnanyouzi/article/details/3721091)

[C++的继承与派生](http://www.cnblogs.com/fzhe/archive/2012/12/25/2832250.html)

PS：

（1）调用虚函数时，因为是动态绑定，所以根据指针指向的对象的实际类型来决定。

（2）调用非虚函数，静态绑定，所以根据表面上看到的类的类型来决定。