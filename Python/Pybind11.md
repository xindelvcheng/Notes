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

PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 example plugin"; // optional module docstring

    m.def("add", &add, "A function which adds two numbers");
}
```
pybind11是纯头文件库，编译时C++端只需将其头文件加入头文件路径，而不需要指定.lib和.dll路径。头文件路径可以通过`python3 -m pybind11 --includes`命令获得。

```bash
$ python3 -m pybind11 --includes
-IC:\Python\Include -IC:\Python\lib\site-packages\pybind11\include
```
这里的-I是gcc的参数（include），因此使用gcc编译的命令为：

```bash
c++ -O3 -Wall -shared -std=c++11 -fPIC $(python3 -m pybind11 --includes) test.cpp -o example$(python3-config --extension-suffix)
```

注意，这里生成的目标文件（在Linux上是一个.so）的名称后缀是不能随意指定的（对Python3而言），否则之后不能直接导入。

### 3.在Python中使用

在.so同级文件夹使用ipython可以直接导入example库并使用函数add。

```python
In [1]: import example
In [2]: print(example.add(1,2))
3
```



