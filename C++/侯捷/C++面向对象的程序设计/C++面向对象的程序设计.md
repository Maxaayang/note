正规的、大气的编程习惯

头文件
```C++
#ifndef _COMPLEX_	//防卫式声明
#define _COMPLEX_

#include <cmath>

class ostream;	//前置声明
class complex;

complex& _dopal (complex* ths, const complex& r);

class complex	//类声明
{
	...
}

complex::function ...	//类定义
#endif
```

模板
```C++
template<typename T>
class complex{
public:
	complex(T r = 0, T i = 0)
		: re(r), im(i)		//初始化（效果更好）
	{
		re = r;				//赋值
		im = i;
	}
	int func(const complex& param)	//相同class的各个objects互为friends
	{
		return param.re + param.im;
	}
	complex& operator += (const complex&);	//参数传递最好使用引用
	T real () const {return re;}	
	T image () const {reaturn im;}
private:
	T re, im;
	
	friend complex& _doapl(complex*, const complex&);
};

{
	complex<double> c1(2.5, 1.5);
	complex<int c2(2, 6);
	...
}

{
	const complex c1(2,1);
	cout << c1.real();		//如果类中输出函数没有加const则无法打印
	cout << c1.imag();
}
```
不带指针的类多半不用写析构函数
1、数据都在private中
2、参数尽量都用引用传递
3、返回值尽量用引用传递，然后考虑使用引用是否会有问题
4、在类里面的函数注意加const
5、构造函数使用初始化，不要使用赋值

inline函数
```C++
inline complex&
_doapl(complex* ths, const complex& r)	//传递者无需知道接收者是以引用形式接收
{
	ths->re += r.re;	//第一参数将会被改变
	ths->im += r.im;	//第二参数不会被改变
	return *ths;
}

inline complex&		//操作符重载
complex::operator += (const complex& r)
{
	return _doapl (this, r);	//this指针
}

//不可以返回return by reference，它们返回的必定是local object
//临时对象 typename()
inline complex 	//非成员函数操作符重载，如果写在类里边，则只能实现复数加复数
operator + (const complex& x, const complex& y)
{
	return complex (real (x) + real (y),
					imag (x) + imag(y));
}

inline complex	//共轭复数
conj (const complex& x)
{
	return complex (real (x), -imag(x));
}

//返回类型为 ostream&，以防连续输出，不要用 void
ostream&	//只能作为全局函数重，返回不可以为const，在函数内部一直在改变os的状态
operator << (ostream& os, const complex& x)
{
	return os << '(' << real (x) << ',' << imag(x) << ')';
}

{
	complex c1(2, 1);
	complex c2(5);
	
	c2 += c1;
}
```

构造函数
```C++
class complex
{
public:
	complex (double r = 0, double i = 0)	//默认实参
		: re (r), im (i)	//初始化（效果好）
	{
		re = r;				//赋值
		im = i;
	}

private:
	double re, im;
}
```
不带指针的类多半不用写析构函数


```C++
class String
{
public:
	String(const char* cstr = 0);
	String(const String& str);	//拷贝构造
	String& operator = (const String& str);	//拷贝赋值
	~String();	//析构函数
	char* get_c_str() const { return m_data; }
private:
	char* m_data;
};

inline
String::String(const char* cstr)
{
   if (cstr) {
      m_data = new char[strlen(cstr)+1];
      strcpy(m_data, cstr);
   }
   else {   //未指定初值
      m_data = new char[1];
      *m_data = '\0';
   }
}

inline
String::~String()	//释放内存
{
   delete[] m_data;
}
```


Stack，是存在于某个作用域的一块内存空间。当你调用函数，函数本身即会形成一个stack用来放置它所接收的参数，以返回地址。在函数本身内声明的任何变量，其所使用的内存都取自上述stack。

Heap，是指由操作系统提供的一块全局内存空间，程序可动态分配从其中获得若干区块。必须手动释放掉内存。
* new 先分配内存，再调用构造函数
* delete 先调用析构函数，再释放内存
```C++
class Complex { ... };
{
	//c1 所占用的空间来自 stack
	//c1 便是所谓 stack object，其生命在作用域（scope）结束之际结束。
	//这种作用域内的 object，又称为 auto object，因为它会被自动清理。
	Complex c1(1, 2);
	//Complex(3) 是个临时对象，其所占用的空间乃是以 new 自 heap 动态分配而得，并由 p 指向
	Complex* p = new Complex(3);
	...
	//必须被释放，否则内存泄漏，p 所指的 heap object 仍然存在，但指针 p 的生命却结束了。
	delete p;	
	//c2 便是所谓 static object，其生命在作用域（scope）结束之后仍然存在，直到整个程序结束。
	static Complex c2(1, 2);
}

//c3 便是所谓 global object，其生命在整个程序结束之后才结束。可以把它视为一种 static object，其作用域是整个程序。
Complex c3(1, 2);

int main()
{
	...
}

{
	String* p = new String[3];
	...
	delete[] p;	//唤起 3 次 dtor
	delete p;	//唤起 1 次 dtor
}
```

静态函数没有 this 指针
调用静态函数的方式：
* 通过object调用
* 通过class name调用
```C++
class Account {
public:
	static double m_rate;
	static void set_rate(const double& x) { m_rate = x;}
};
double Account::m_rate = 8.0;

int main() {
	Account::set_rate(5.0);
	
	Account a;
	a.set_rate(7.0);
}
```


复合
```C++
template <class T>
class queue {
	...
protected:
	deque<T> c;	//底层容器
public:
	//以下完全利用 c 的操作函数完成
	bool empty() const { return c,empty(); }
	size_type size() const { return c.size(); }
	reference front() { return c.front(); }
	reference back() { return c.back(); }
	void push(const value_type& x) { c.push_back(x); }
	void pop() { c.pop_front(); }
};
```
构造由内而外，析构由外而内。

有外部就有内部



委托
```C++
class StringRep;
class String {
public:
	String();
	String(const char* s);
	String(const String& s);
	String &operator=(const String& s);
	~String();
...
private:
	StringRep* rep;	//pimpl 有一个指针，随时可以调用 StringRep；编译防火墙
};

namespace {
class StringRep{
friend class String;
	StringRep(const char* s);
	~StringRep();
	int count;
	char* rep;
};
}

String::String(){ ... }
```

两个函数生命周期不同步。



继承

```C++
struct _List_node_base
{
	_List_node_base* _M_next;
	_List_node_base* _M_prev;
};

template<typename _Tp>
struct _List_node
	: public _List_node_base	//继承
{
	_Tp _M_data;
};
```
构造由内而外：先调用父类的默认函数，再执行自己。

析构由外而内：先执行自己，再调用父类的析构函数。

​							父类的析构函数必须是 virtual，否则会出现 undefined behavior



虚函数与多态

virtual，override（虚函数被重新定义）

non-virtual 函数：不希望 derived class 重新定义（override，覆写）它。

virtual 函数：希望 derived class 重新定义（override，覆写）它，且对它已有默认定义。

pure virtual 函数：希望 dived class 一定要重新定义（override，覆写）它，对它没有默认定义。

函数继承的是调用权

```C++
class Shape{
public:
  virtual void draw() const = 0;	//pure virtual
  virtual void error(const std::string& msg);	//impure virtual
  int objectID() const;	//non-virtual
  ...
};

class Rectangle:public Shape{ ... };
class Ellipse:public Shape{ ... };
```

纯虚函数子类一定要覆写，空函数不一定要覆写















