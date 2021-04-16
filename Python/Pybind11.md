# Pybind11

[TOC]

Pybind11用于Python调用C++代码。

## 一.安装Pybind11

### 1.python端

```python
$ pip install pybind11
```

### 2.C++端

写一个简单的名为example的库，其中含有一个名为add的方法。

```c++
#include <pybind11/pybind11.h>

int add(int i, int j) {
    return i + j;
}

//创建一个名为example的库，注意不需要是字符串
PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 example plugin";

    m.def("add", &add, "Add two numbers.");
}
```
pybind11是纯头文件库，编译时C++端只需将其头文件加入头文件路径，而不需要指定.lib和.dll路径。头文件路径可以通过`python3 -m pybind11 --includes`命令获得。

##### ②源码编译pybind11测试用例
```bash
$ python3 -m pybind11 --includes
-IC:\Python\Include -IC:\Python\lib\site-packages\pybind11\include
```
这里的-I是gcc的参数（include），因此使用gcc编译的命令为：
```bash
c++ -O3 -Wall -shared -std=c++11 -fPIC $(python3 -m pybind11 --includes) test.cpp -o example$(python3-config --extension-suffix)
```

注意，这里生成的目标文件（在Linux上是一个.so）的名称后缀是不能随意指定的（对Python3而言），否则之后不能直接导入。

使用CMake时，可以find_package（适用于pip或make安装的情况）或add_subdirectory(extern/pybind11)（适用于仅源码的情况），并使用得到的函数pybind11_add_module（包装了add_library）创建.lib。需注意设定toolchain为64bit，同时需要选择“编译项目”，因为library不会出现在单个编译菜单中。

```cmake
find_package(pybind11 REQUIRED)
pybind11_add_module(example example.cpp)
```



### 3.在Python中使用

在.so同级文件夹使用ipython可以直接导入example库并使用函数add。

```python
In [1]: import example
In [2]: print(example.add(1,2))
3
```

## 二.使用pybind11

### 1.导出变量

内建变量和stl提供中的一些类的对象可以使用m.attr直接导出，如下：

```c++
PYBIND11_MODULE(example, m) {
    m.attr("the_answer") = 42;
}
```

### 2.导出函数

函数关键字参数和默认值可以使用pubind11::arg()方法将元信息传递给m.def()，如下：

```c++
int add(int i, int j) {
    return i + j;
}

m.def("add", &add, "Add two numbers.",py::arg("i") = 1, py::arg("j") = 2);
```

这里的函数参数来自Python，Python、

### 3.导出类

使用pybind11::class_创建类绑定。

```c++
struct Pet {
  Pet(const std::string &name) : name(name) { }
  void setName(const std::string &name_) { name = name_; }
  const std::string &getName() const { return name; }

  std::string name;
};

namespace py = pybind11;
PYBIND11_MODULE(example, m) {
  py::class_<Pet>(m,"Pet")
      .def(py::init<const std::string&>())
      .def("setName",&Pet::setName)
      .def("getName",&Pet::getName);
}
```

使用：

```python
In [1]: import example

In [2]: pet = example.Pet('Hello')

In [3]: print(pet)
<example.Pet object at 0x0000022E52DFB170>

In [4]: pet.getName()
Out[4]: 'Hello'

IIn [5]: pet.setName('nihao')

In [6]: pet.getName()
Out[6]: 'nihao'
```

这里因为`__repr__`和`__str__`都没有重写的原因（实际上pybind11中`__str__`不起作用），打印对象返回的是一个没什么意义的消息，因此添加`__repr__方法` ，如下：

```c++
py::class_<Pet>(m,"Pet")
    .def(py::init<const std::string&>())
    .def("setName",&Pet::setName)
    .def("getName",&Pet::getName)
    .def("__repr__",[](const Pet& a){
        return "<example.Pet named '" + a.name + "'>";
    });
```

类中的方法有一个隐式的参数this或self，在pybind11中这里既可以是引用也可以是指针，下面的也可以工作：

```c++
.def("__repr__",[](Pet* a){
        return "<example.Pet named '" + a->name + "'>";
    })
```



### 4.导出属性

使用def_readwrite()或def_readonly()添加属性，如下:

```c++
py::class_<Pet>(m,"Pet")
    .def(py::init<const std::string&>())
    .def_readwrite("name",&Pet::name)
    ;
```

使用时：

```python
pet.name = 'pet'
```

如果该属性是一个私有变量，则可以使用def_property和def_property_readonly将对该变量的读写绑定到getter/setter上，如下：

```c++
class Pet {
  std::string name;
 public:
  explicit Pet(const std::string &name) : name(name) {}
  void setName(const std::string &name_) { name = name_; }
  const std::string &getName() const { return name; }

};

namespace py = pybind11;
PYBIND11_MODULE(example, m) {
  py::class_<Pet>(m, "Pet")
      .def(py::init<const std::string &>())
      .def_property("name", &Pet::getName, &Pet::setName);
}
```

> 如果def_property中getter/setter传入的是nullptr，将会创建一个只写/只读函数。

### 5.动态属性

从C++导出的类默认不像原生的Python类那样支持动态添加属性，例如

```python
class Pet():
    def __init__(self, name):
        self.name = "name"
        
pet = Pet()
pet.name = "Hello"
pet.age = 2 # OK

pet = example.Pet()
pet.name = "Hello"
pet.age = 2 # Error
```

如果要支持动态属性，需要在py::class_中添加py::dynamic_attr()标签作为参数。

```c++
py::class_<Pet>(m, "Pet",py::dynamic_attr())
    .def(py::init<const std::string &>())
    .def_property_readonly("name", &Pet::getName);
```

### 6.继承和自动下转

C++中的继承关系不能被自动导出（因为这本来就是编译器的工作？），如果要导出继承关系需要手动在py::class_中的模板变量中指定，如下：

```c++
py::class_<Dog, Pet>(m, "Dog")
```

如果Pet类是一个多态类（即含有virtual方法），那么如果从C++返回一个Pet指针指向的Dog类对象，将会被pybind11自动转成Dog类型，如下：

```c++
class Pet {
  std::string name;
 public:
  explicit Pet() = default;
  virtual ~Pet() = default;
};

class Dog : public Pet {
 public:
  explicit Dog() = default;
};

namespace py = pybind11;
PYBIND11_MODULE(example, m) {
  py::class_<Pet>(m, "Pet")
      .def(py::init());

  py::class_<Dog, Pet>(m, "Dog")
      .def(py::init());

  m.def("get_dog_object", []() {
    return std::unique_ptr<Pet>(new Dog());
  });
}
```

使用时：

```python
In [1]: import example

In [2]: obj = example.get_dog_object()

In [3]: type(obj)
Out[3]: example.Dog
```

### 7.函数重载

因为重载时函数名具有多义性，因此需要指定绑定函数指针的类型或使用py::overload_cast（适用于C++14及以上）或py::detail::overload_cast_impl（适用于C++11）指定函数输入参数让pybind推导函数指针类型，如下：

```c++
class Pet {
  std::string name_;
  int age_;
 public:
  explicit Pet() = default;
  virtual ~Pet() = default;

  void set(const std::string &name) { name_ = name; }
  void set(int age) { age_ = age; }
};

namespace py = pybind11;
PYBIND11_MODULE(example, m) {
  py::class_<Pet>(m, "Pet")
      .def(py::init())
      .def("set", py::overload_cast<const std::string &>(&Pet::set), "set name")
      .def("set", py::overload_cast<int>(&Pet::set), "set age");
}
```

特别地，重载的构造函数只需要一个接一个地使用.def(py::init<...>())语法声明即可。

也可以使用lambda调用其中一个函数。

### 8.枚举

枚举需要使用py::enum

```
struct Pet {
    enum Kind {
        Dog = 0,
        Cat
    };

    Pet(const std::string &name, Kind type) : name(name), type(type) { }

    std::string name;
    Kind type;
};
```

绑定：

```c++
py::class_<Pet> pet(m, "Pet");

pet.def(py::init<const std::string &, Pet::Kind>())
    .def_readwrite("name", &Pet::name)
    .def_readwrite("type", &Pet::type);

py::enum_<Pet::Kind>(pet, "Kind")
    .value("Dog", Pet::Kind::Dog)
    .value("Cat", Pet::Kind::Cat)
    .export_values(); // 导出字典到Pet.Kind.__members__，对enum class省略此行
```

注意，这里需要一个class instance传给py_enum。

