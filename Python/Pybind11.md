# Pybind11

[TOC]

Pybind11用于Python调用C++代码。

## 一.安装Pybind11

### 1.python端

```python
$ pip install pybind11
```

### 2.C++端

##### ①获得pybind11源码

```bash
$ git clone git@github.com:pybind/pybind11.git
```

##### ②源码编译pybind11测试用例

```bash
$ cd pybind11
$ cmake -S . -B build
$ cmake --build build -j 2  # Build on 2 cores
$ cmake --install build # need admistrator
$ python3 setup.py install
```

##### ③引入pybind11

pybind11是纯头文件库，C++端只需将其头文件加入头文件路径，而不需要指定.lib和.dll路径：

```cmake
include_directories("C:/Program Files/Python37/Lib/site-packages/pybind11/include")
```

