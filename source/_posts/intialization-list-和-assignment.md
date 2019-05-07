title: intialization list 和 assignment
author: Nico
tags:
  - C++
categories:
  - C++
date: 2019-05-07 22:32:00
---
	intialization list指的是初始化列表.
	Assignment 指的是赋值.
   
 #### 什么是初始化列表
 
 构造函数除了有名字，参数列表和函数体之外，还可以有初始化列表，初始化列表以冒号开头，后跟一系列以逗号分隔的初始化字段。
 
 
 ```
 class foo
{
    string name;
    int id;
    foo(string s, int i):name(s), id(i){} ; //初始化列表
};
 ```
 
 #### 构造函数的两个执行阶段：

构造函数的执行可以分为两个阶段，初始化阶段和计算阶段。初始化阶段先于计算阶段。

###### 初始化阶段：

所有类类型的成员都会在初始化阶段被初始化，即使该成员没有出现在构造函数的初始化列表中.

###### 计算阶段：

一般用于执行构造函数体内的赋值操作.

下面代码定义两个类，Test1有构造函数，拷贝构造函数及赋值运算符。（为方便查看结果）

Test2是测试类，它以Test1的对象为成员.

```
#include<iostream>
using namespace std;
 
class Test1
{
    public:
        Test1()
        {
            cout << "Construct Test1" << endl;
        }
        
        Test1(const Test1& t1)
        {
            cout << "Copy Construct for Test1" << endl;
            this->a = t1.a; 
        }
        
        Test1& operator = (const Test1& t1)
        {
            cout << "assignment for Test1" << endl;
            this->a = t1.a;
            return *this;
        }
        
    private:    
        int a;
};
 
class Test2
{
    public:
        Test2(Test1& t1)
        {
            test1 = t1;
        }
        
    private:
        Test1 test1;
 
};
 
int main()
{
    Test1 t1;
    Test2 t2(t1);
    
    return 0;
}
```
```
Construct Test1
Construct Test1
assignment for Test1

```

结果解释：

第一行输出对应调用代码中第一行，构造一个Test1对象。

第二行输出对应Test2构造函数中的代码，用默认的构造函数初始化对象test1，这就是所谓的初始化阶段。

第三行输出对应Test1的赋值运算符中代码，对test1执行赋值操作，这就是所谓的计算阶段。


#### 为什么使用初始化列表：

初始化类的成员有两种方式：

- 使用初始化列表
- 在构造函数体内进行赋值

主要是基于一个性能的问题.对于内置类型（int,float,double…），使用初始化列表和在构造函数体内初始化，差别不大。但是对于类类型来说，最好使用初始化列表.


修改上面的代码：(只修改了Test2这个类)
```
class Test2
{
	public:
		Test2(Test1& t1):test1(t1) 
		{
		}
		
	private:
		Test1 test1;

};
```

```
Construct Test1
Copy Construct for Test1

```
结果解释：

第一行输出对应 main函数的第一行.

第二行输出对应Test2的初始化列表，直接调用拷贝构造函数初始化test1，省去了调用默认构造函数的过程。所以一个好的原则是：能使用初始化列表的时候尽量使用初始化列表.

#### 哪些情况下必须使用初始化列表：

除了前面提到的性能之外，在部分情况下是必须使用初始化列表的：

- 常量成员.因为常量只能初始化而不能赋值，所以必须使用初始化列表
- 引用类型.必须在定义的时候初始化，并且不能重新赋值，所以要写在初始化列表里面.
- 没有默认构造函数的类类型.因为使用初始化列表可以不必调用默认的构造函数来初始化，而是直接调用拷贝构造函数来初始化.

对于没有默认构造函数的例子

```

#include <iostream>
using namespace std;

class Test1
{
	public:
		Test1(int a):a(a)
		{
		}
		
	private:
		int a;
};

class Test2
{
	public:
		Test2(Test1 &t1)
		{
			test1 = t1;
		}
	private:	
		Test1 test1;
};

int main()
{
	Test1 t1(1);
	Test2 t2(t1);
	return 0;
}
```
以上代码无法通过编译，因为Test2的构造函数中test1 = t1这一行实际上分成两步执行。

- 调用Test1的默认构造函数来初始化test1
- 调用Test1的赋值运算符给test1赋值

但是由于Test1没有默认的构造函数，所谓第一步无法执行，故而编译错误。

正确的代码如下，使用初始化列表代替赋值操作:

```
class Test2
{
	public:
		Test2(Test1 &t1):test1(t1) 
		{
		}
	private:	
		Test1 test1;
};
```

#### 成员变量的初始化顺序：
成员是按他们在类中出现的顺序初始化的.而不是按初始化列表的顺序.

```
#include <iostream>
using namespace std;

class Type
{
	public:
		int i;
		int j;
	
		Type(int x):i(x),j(i)
		{
		}
};

int main()
{
	Type type(1);
	cout << type.i << type.j << endl; 
}
```

试比较上下两段代码的输出结果：

```
#include <iostream>
using namespace std;

class Type
{
	public:
		int i;
		int j;
	
		Type(int x):j(x),i(j)
		{
		}
};

int main()
{
	Type type(1);
	cout << type.i << type.j << endl; 
}
```

上面代码的输出结果是：11。

下面代码的输出结果是：01。

结果解释：

这里i的值是未定义的（默认为0）因为虽然j在初始化列表里面出现在i前面，但是i先于j定义。

所以先初始化i，但i由j初始化，此时j尚未初始化，所以导致i的值为默认的0。

所以可以看出，是按定义顺序而不是初始化列表的顺序进行初始化的。

所以，一个好的习惯是，按照成员定义的顺序进行初始化。
