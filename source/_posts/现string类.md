title: 实现string类及右值和左值
author: Nico
tags:
  - C++
categories:
  - C++
date: 2019-05-06 21:37:00
---
```
class string
{
public：
  String()    //初始化
  : _pstr(new char[1])
  {}
 
String(const char * pstr );     //普通构造函数
: _pstr(new char[strlen(pstr) + 1]())
{
　　strcpy(_pstr,pstr);
}
 
String(const String & rhs); 	//复制构造函数（深度拷贝）
: _pstr(new char[strlen(pstr) + 1]())
{
　　strcpy(_pstr, rhs.pstr);
}
 
String(String && rhs);  //移动构造函数,右值引用
: _pstr(rhs._pstr)
{
　　rhs.pstr = NULL;
}
 
String & operator=(const String & rhs)  //重载赋值运算符函数
{
　　if(this != & rhs)
　　{
　　　　delete [] _pstr;
　　　　_pstr = new char[strlen(rhs._pstr) + 1]();
　　　　strcpy(_pstr, rhs._pstr);
　　}
　　return *this;
}
String & operator=(String && rhs) //移动赋值运算符函数
{
　　if(this != &rhs)
　　{
　　　　delete [] _pstr;
　　　　_pstr = rhs._pstr;
　　　　rhs._pstr = NULL;
　　}
　　return * this;
}
 //析构函数
~String()
{
　　delete [] _pstr;
}
 	//重载输出流运算符友元函数
　　friend std::ostream &operator<<(std::ostream & os, const String & rhs);
  
private:
　　char * _pstr;//私有数据成员
};
 
std::ostream & operator<<(std::ostream & os, const String & rhs)
{
　　os << rhs._pstr;
　　return os;
}
```

#### 注意

```
void f_ck(int & i) {
    i++;
}
...
    f_ck(1); // 编译不通过，VS答曰：非常量的引用值必须是左值。
...
```

	& 是左值引用	，所谓左值就是实际在运行时内存中存在的值，而不是 1 这种写死在运行代码中的值。
    
#### 返回引用的生命周期问题
```
#include <iostream>
#include <string>

using namespace std;

class Good {
public:
    string goodname;
    Good() {
        goodname = "not_good";
        cout << "Good constructed" << endl;
    }
    ~Good() {
        cout << "Good destoyed" << endl;
    }
};

class Warehouse {
public:
    Good & good;
    Warehouse(Good & good) : good(good)
    {

    }
    //错误将栈变量赋值给左值引用，栈的生命周期要小于左值引用
    static Warehouse* WarehouseBuilder() {
        Good good;
        return new Warehouse(good);
    }
};

int main()
{   
    Warehouse * warehouse = Warehouse::WarehouseBuilder();
    cout << warehouse->good.goodname << endl;
    getchar();
    return 0;
}

```

  	&&，就是右值引用了，右值是出现在等式右边的值。 
  	右值 && 显然不能放在等号左边。
    
```
class Whore {
public:
    string nickname;
};

class Brothel {
public: 
    Whore && whore_;
    Brothel(Whore && whore): whore_(whore) { //编译不通过，无法将右值引用绑定到左值
    }
	//whore_=whore 不能
};
```
相似的
```
class Whore {
public:
    string nickname;
};

class Brothel {
public: 
    Whore && whore_;
    Brothel(Whore && whore) { // 编译不通过，“Brothel::whore_”: 必须初始化引用
        whore_ = whore;
    }
};
```

```
class Whore {
public:
    string nickname;
};

class Brothel {
public: 
    Whore & whore_;
    Brothel(Whore && whore): whore_(whore) {
    }

};

...
    Whore && whore = Whore();
    Brothel(whore); // 编译不通过
...
```

	用 type && r 去引用一个匿名的右值（右值当然是匿名的），确实延长了其生命周期，但是这有个限度，就是不能超过栈的生命周期。
    
    c++ 中的左值就像指针，它可以捕获实实在在的实体，但是我们要注意被捕获值的生命周期。不要随便把生命周期和栈同步的实体传给了它； 
	右值其实也是指针，但是它功能是专门捕获匿名的实体（可以理解为编译产生的中间变量）。同时我们要注意，右值在定义时捕获了实体以后，右值的名字就变成了被捕获的实体。
    
    一个左值表达式代表的是对象本身，而右值表达式代表的是对象的值；变量也是左值。
    
    
   - 绑定的对象（引用的对象）不同，左值引用绑定的是返回左值引用的函数、赋值、下标、解引用、前置递增递减
   
- 左值持久，右值短暂，右值只能绑定到临时对象，所引用的对象将要销毁或该对象没有其他用户

- 使用右值引用的代码可以自由的接管所引用对象的内容

```
int i = 0;  // 在这条语句中，i 是左值，0 是临时值，就是右值。
```

```
int &a = 1;   // error C2440: “初始化”: 无法从“int”转换为“int &”
1
我们最多只能用常量引用来绑定一个右值，如：
```

在C++11中，我们可以引用右值，使用&&来实现：
```
int &&a = 1;
```

	临时对象被使用完之后会被立即析构，在析构函数中free掉申请的内存资源。 如果能够直接使用临时对象已经申请的资源，并在其析构函数中取消对资源的释放，这样既能节省资源，有能节省资源申请和释放的时间。
    
    右值引用并不能阻止编译器在临时对象使用完之后将其释放掉的事实，所以转移构造函数和转移赋值操作符重载函数 中都将_data赋值为了NULL，而且析构函数中保证了_data != NULL才会释放。
  	 标准库提供了函数 std::move，这个函数以非常简单的方式将左值引用转换为右值引用。
    
```
template <class T> 
void swap(T& a, T& b) 
{ 
    T tmp(a);   // copy a to tmp 
    a = b;      // copy b to a 
    b = tmp;    // copy tmp to b 
}
```
```
template <class T>
void swap(T& a, T& b) 
{ 
    T tmp(std::move(a)); // move a to tmp 
    a = std::move(b);    // move b to a 
    b = std::move(tmp);  // move tmp to b 
}
```