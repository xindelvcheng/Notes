# C++随笔

[TOC]

##### 1.创建一个对象申请的栈/堆空间只包括属性，不包括方法（C++将类设计为一个struct+一堆含有this参数的普通函数）

##### 2.重载只能发生在一个类中，子类不能重载父类的方法；若子类的方法与父类同名，将覆盖父类的方法（此时C++编译器只会在子类中查找该函数的重载），若要调用父类方法需使用域作用符指明为父类方法。若子类方法与父类方法签名相同时，称为重写，若方法为virtual，则为多态。

##### 3.const只会组织修改一个对象，但不会阻止修改对象中指针指向的另外对象的值

##### 4.之所以在构造函数中调用虚函数不会表现出多态，是因为vptr指针的初始化是分步骤的，当子类调用构造函数时会先调用父类构造函数，此时子类的vptr指向父类的虚函数表，到执行子类构造函数时vpt指向子类的虚函数表

##### 5.C++中模板的实现就是根据实际使用的类型生成普通函数和普通类，例如func\<int\>()、func\<char\>将会生成两份func的代码（在汇编中可以看到）。实际上，函数模板将会经过两次编译，函数模板本身编译一次，在调用的地方对参数替换后的代码再进行依次编译

```bash
$ gcc test.cpp -S -o test.s
```

因此，模板参数不同的函数或类之间的静态变量并不共享：

```c++
#include <cstdio>
#include <vcruntime_typeinfo.h>

template<typename T>
void static_func() {
    static int i = 10;
    printf("static_func<%s>:%d\n",typeid(T).name(), i++);
}

int main() {
    static_func<int>();
    static_func<int>();
    static_func<float>();
}
```

输出：

```
static_func<int>:10
static_func<int>:11
static_func<float>:10
```

##### 6.友元函数不能滥用，遇到模板会出现bug（声明定义分离时因为模板二次编译的机制造成两个函数头不同而编译失败），只在重载<<和>>运算符时使用

##### 7.模板尽量不要声明与定义分离，会很麻烦（在类外部模板都需要使用T实例化，而言使用时需要引入包含实现的.cpp而不是模板声明的.h），老编译器也不一定支持	

##### 8.const_cast一般用于c语言中隐式类型转换的地方不能任意转换对象只能转换有继承关系的指针（但不会进行类型检查），dynamic_cast一般用于泛型的父类指针转子类指针附带类型检查（如果失败将会返回空指针，但要求该类须有虚函数），reinterpret_cast可以无限制地进行指针转换；const_cast用于去除变量的只读属性

```c++
class BaseClass {
public:
    int a = 0;
    virtual void AnyFunc() {};
};

class DerivedClass : public BaseClass {
public:
    int b = 0;
};

class OtherClass {
};

int main() {
    BaseClass *base = new BaseClass();

    //使用static_cast转换不会类型检查，但是若使用返回的Derive指针调用
    DerivedClass *Derive = static_cast<DerivedClass *>(base);
    Derive ? printf("Derive static_cast success\n") : printf("Derive static_cast fail\n");
    printf("Derive->a:%d\n",Derive->a);
    printf("Derive->b:%d\n",Derive->b); /*未定义行为，将访问不属于该对象的内存，因为不知道是谁的，结果也是不确定的*/

    DerivedClass *Derive2 = dynamic_cast<DerivedClass *>(base);
    Derive2 ? printf("Derive2 dynamic_cast success\n") : printf("Derive2 dynamic_cast fail\n");

    OtherClass *other = reinterpret_cast<OtherClass *>(base);
    
    //注意不能使用char*，否则字符将在只读数据区，无论string是不是const都不能修改（语法可以，运行报错）
    const char string[] = "Hello!";
    //string[5] = "." /*报错*/
    char *mutable_string = const_cast<char *>(string);
    mutable_string[5] = '.';
    printf("%s\n", mutable_string);
}
```

输出：

```c++
Derive static_cast success
Derive->a:0
Derive->b:-33686019
Derive2 dynamic_cast fail
Hello.
```

##### 9.OpenMP一定需要手动启动

①VS：项目属性→语言→OpenMP

②gcc：-fopenmp

③cmake:

```
find_package(OpenMP)
target_link_libraries(example-app PRIVATE OpenMP::OpenMP_CXX)
```

对老版本CMake

```
FIND_PACKAGE( OpenMP REQUIRED)
if(OPENMP_FOUND)
    message("OPENMP FOUND")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")
endif()
```