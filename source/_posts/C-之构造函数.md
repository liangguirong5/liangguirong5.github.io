---
title: C++之构造函数详解
tags:
  - C++
categories: C++
date: 2017-02-05 22:24:23
---

# 构造函数

当Richard类的对象被创建时，自动调用相应的Richard()构造函数，对成员变量进行初始化，这就是构造函数。

<!-- more -->

```c++
class Richard()
  {
    public:
  		Richard()
          {
            m_val=1;
          }
  	private:
  		int m_val;
  }
```



# 构造函数的分类

构造函数可以分为无参数构造函数、一般构造函数、拷贝构造函数、类型转换构造函数。

```C++
class Complex 
{         

private :
        double    m_real;
        double    m_imag;
public :
		无参数；
		一般；
		拷贝构造；
		类型转换；
		等号运算符重载(不属于构造函数)
}
```

## 无参数构造函数

如果在一个类中没有写明构造函数，系统会自动生成无参数构造函数，函数为空，什么都不做。

```C++
Complex()
{
	m_real = 0.0;
	m_imag = 0.0;
} 
```

## 一般构造函数

一般构造函数可以有各种参数形式，一个类可以有多个一般构造函数，前提是参数的个数或者类型不同（基于c++的重载函数原理）

```C++
Complex(double real, double imag)
{
	m_real = real;
	m_imag = imag;
} 
```

## 拷贝构造函数

拷贝构造函数是类对象本身的引用，用于根据一个已存在的对象复制出一个心得该类对象，一般在函数中会将已存在的对象数据成员的值复制一份，到新创建的对象中。

若果没有显示的写出拷贝构造函数，系统会创建一个默认的，但是这是会造成浅拷贝，如需深拷贝则需自己编写。

```c++
Complex(const Complex& c)
  {
    m_real = c.m_real;
    m_imag = c.m_imag;
  }
```

注意：等号运算符重载和拷贝构造函数是有区别的，将=右边的本类对象的值复制给等号左边的对象，它不属于构造函数，等号左右两边的对象必须已经被创建。

```c++
Complex& operator=(const Complex& rhs)
  {
    if(this==&rhs)
      {
        return *this;
      }
  	this->m_real = rhs.m_real;
  	this->m_imag = rhs.m_imag;
  	return *this
  }
```

## 类型转换构造函数

根据一个指定的类型的对象创建一个本类的对象，例如：下面将根据一个double类型的对象创建了一个Complex对象。

```C++
Complex(double r)
{
    m_real = r;
    m_imag = 0.0;
}
```

类型转换构造函数，只能有一个参数，该参数为待转换的类型。下面举一个例子，执行语句

```c++
Complex c2;
c2 = 5.2;
```

第一行调用无参数构造函数；第二行中，先对5.2进行doule到complex的强制转换，也就是调用类型转换构造函数，然后在调用等号赋值。

# 参考阅读

[c++类的构造函数详解](http://blog.csdn.net/tiantang46800/article/details/6938762)

