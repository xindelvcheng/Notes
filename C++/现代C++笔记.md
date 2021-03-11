# Modern C++ Notes

[TOC]

C++  is not C with Class,It is another language.

You should never worry about the performance of your program unless you must to do it.

## class definition

Follow a example of A typical class definition.

##### Example Person.h

Every .cc file should have its .h file. Because c++ program lost most meta-information after compile,while java & C# still remain all meta-information in their .java & .exe/.dll file so they can called without a extra declare of what method and field in this program.

> By the way ,some dll may remain a list of function offset in their 'PE head',so you can use them without a .h file,but the .exe do not make a commitment to this,so you need a .h to use the binary file of c/c++.

```c++
//
// Created by 26046 on 6/6/2020.
//

#ifndef TEST_PERSON_H
#define TEST_PERSON_H


#include <stdio.h>
#include <string>

/*1.Do not use using namespace to pollute the current namespace;*/
using std::string;

/*2.Unnamed namespace is recommended to seperate symbols,don not add a indent for the content of the namespace;*/
/*Of Course,if you use the unnamed namespace ,you can not find them in other file because the compile name them in a */
/*symbol you do not know just like 'namespace __UNIQUE_NAME_ {};using namespace __UNIQUE_NAME_;'*/
/*中文释义：只能使用public继承；若使用匿名命名空间，其内容在本文件中直接使用（实际上相当于把匿名命名空间的中内容移到一个__UNIQUE_NAME_命名空间中，再在本文件加上一个using namespace __UNIQUE_NAME_,确保本文件可访问），在其他文du件中无法直接使用该匿名命名空间的内容。*/
namespace pawn {
class Person {
public:
    Person(string name, int age);

    /*3.Method param
		①If you do know,use 'const T&'
        ②else ,use 'p*'
        That is to say,You should not choose 'const *'or '&' if you have no idea.
    */
    virtual void sayHello(const string &);
    

    void run();

    /*Disable the copy and move function if you do not need them;*/
    Person(const Person &) = delete;
    Person &operator=(const Person &) = delete;

    /*Use method overload rather than default param,do not use define a lot of method overload rather than make its name clear*/
    void eatFood();
    void eatCake();


private:
    string name;
    int age;
};


class Greet {
public:
    virtual void run() = 0;
};

}//namespace pawn
#endif //TEST_PERSON_H

```

##### Example Person.cpp

```c++
//
// Created by 26046 on 6/6/2020.
//

#include "Person.h"
namespace pawn {
Person::Person(string name, int age) {
    this->name = std::move(name);
    this->age = age;
};

void Person::sayHello(const string &) {
    /*4.Only use 'stream' in logging.Use 'printf' in other case.*/
    printf("hello\n");
}

void Person::eatFood() {
    printf("%s\tEat food",this->name);
};

void Person::eatCake() {
    printf("%s\tEat Cake",this->name);
};

void Person::run() {
    printf("I am Running!");
}
}//namespace pawn


```

##### Example Unit Test	  	

```c++
int main(){
    auto p = new pawn::Person("hello!",10);
    p->run();
}
```

## Smart Pointer

>所有权是一种登记／管理动态内存和其它资源的技术. 动态分配对象的所有主是一个对象或函数, 后者负责确保当前者无用时就自动销毁前者. 所有权有时可以共享, 此时就由最后一个所有主来负责销毁它. 甚至也可以不用共享, 在代码中直接把所有权传递给其它对象.

> 智能指针是一个通过重载 `*` 和 `->` 运算符以表现得如指针一样的类. 智能指针类型被用来自动化所有权的登记工作, 来确保执行销毁义务到位. [std::unique_ptr](http://en.cppreference.com/w/cpp/memory/unique_ptr) 是 C++11 新推出的一种智能指针类型, 用来表示动态分配出的对象的独一无二的所有权; 当 `std::unique_ptr` 离开作用域时, 对象就会被销毁. `std::unique_ptr` 不能被复制, 但可以把它移动（move）给新所有主. [std::shared_ptr](http://en.cppreference.com/w/cpp/memory/shared_ptr) 同样表示动态分配对象的所有权, 但可以被共享, 也可以被复制; 对象的所有权由所有复制者共同拥有, 最后一个复制者被销毁时, 对象也会随着被销毁.

> 创建智能指针应该使用`auto p = std::make_queue<ClassName>(ConstructorParams);`的形式，而尽量不要使用`auto p = std::unique_ptr<int[]>(new int[1])`的形式。

1.memory allocation is not only dynamic allocation,consider to stack allocation. If you find that it is necessary to allocation on heap,use smart pointer.

2.method parameter:*smart Pointer (A reference),not smart pointer itself.

示例

```c++
void sortVector(const vector<int>& array) {/*所有按引用传递的参数必须加上 const*/
    std::sort(array->begin(), array->end());
}

int main() {
    auto array = std::make_unique<vector<int>>();
    /*5.Use ++i,not i++*/
    for (int i = 0; i < 10; ++i) {array->push_back(rand());}
    for (auto e : *array) {printf("%d\t",e);}
    printf("\n")
    
    sortVector(*array);//pass a reference,not smart pointer itself.
    
    for (auto e : *array) {printf("%d\t",e);}
}
```

## constepr

true const,can use in array;

```c++
constexpr double pi = 3.14;
```

## int

只使用`int`和`int64_t`。不要使用无符号数。

```c++
int normalNumber = 1000000;
int64_t bigNumber = 10000000000;
```

## Macro

Do not use it if you do not know and do not need. Except #define.

## Auto

You can use in most local variable.

```c++
for (auto e : *array) {printf("%d\t",e);}
```

## Initializer List

Do not use initializer list with auto.

```c++
auto p = new vector<string>{"foo", "bar"}	
```

## Lambda Expression

If you need outer variables you should declare.

```c++
std::sort(v.begin(), v.end(), [](int x, int y) {
    return Weight(x) < Weight(y);
});
```

## Exception

Do not use it.

## Name

Do not use abbreviations except i,T,DNS and so on.