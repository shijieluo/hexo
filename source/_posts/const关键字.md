---
title: const关键字
date: 2018-07-01 20:33:57
categories: C++
tags:
---
# const对象，const指针，const成员函数解析
## const对象
包括内置类型和类的对象，可以定义或声明为如下形式

```
const int num = 10;  //两种写法都ok
int const num = 10;  //
```
## const指针
### 一般情况下的const指针

```
class B{
private:
    int a;
public:
   B(int a=0):a(a){};  
};
const B *pb = B(10); //指针指向B类型的常量；指针所指的对象可以变化，但不可
                    //通过指针改变对象的内容
B * const pb = B(10);//指针所指的对象不可变化，但可以通过指针修改对象
const B * const pb = B(10);//指针所指的对象不可变化，不可以通过指针修改对象
```
### STL迭代器中的const指针
STL迭代器实际上是使用C++指针实现的。所以有：

```
const vector<int>::iterator it = v.begin();
*it = 10;   //正确，相当于定义了一个 T* const 型指针。其所指的值可以改变
++it;       //错误指针所指的对象不可变化
...
vector<int>::const_iterator it = v.begin();
*it = 10;   //错误，相当于定义了一个 const T*型指针，不可修改其所指的值
++it：      //正确，可以修改指针所指的对象
```

## const成员函数
将类的成员函数的加上const关键字，使得const对象的non-static成员变量不发生任何改变。有如下性质：
* 常量对象只能调用它的常量成员函数，而不能调用普通成员函数;
* 普通对象既可以调用常量成员函数，也可以调用普通成员函数;
* 普通成员函数可以访问本类的常量成员函数
* 常量成员函数不能访问本类的普通成员函数;
* 如果常量成员函数与普通成员函数同名，即构成了重载成员函数，那么，常量对象调用常量成员函数，普通对象调用普通成员函数。
另外，当成员函数有const版本和non-const版本时，为了避免代码重复，可以让non-const成员函数调用其const版本。例如

```
class TextBook{
    const char& operator[](std::size_t position) const;
    char& operator[](std::size_t position);
};

char& TextBook::operator[](std::size_t position){
    return const_cast<char&>(static_cast(const TextBook&)(*this)[position]);
}
```
第一个静态转换是为了将non-const对象转换为const对象，否则将会递归调用operator[];第二个转换是去除const属性得到正确的返回值。
## mutable关键字
利用mutable关键字可以使得const成员函数具有改变对象的成员变量的能力。例如：

```
class A{
    mutable int a，b;
public:
    void modifyObject(const int &a, const int &b) const{
        this.a = a;
        this.b = b;
    }    
}
```
这样做是OK的，因为将变量a,b定义为mutable后是可以改变的
