# CMake笔记

[TOC]

## 一.入门

### 1.引言

C/C++的编译历史经过了Manual、Make、CMake三个阶段。

最早，编译一个程序需要两个个步骤：使用编译器将每个源程序编译成.o，使用链接器将.o程序们连接起来。

Make：自动化编译和连接过程，但是需要在Makefile中写每个源文件、头文件、使用的编译器。

CMake：扫描文件夹生成Makefile，但是还是需要编写CMakeLists.txt指明编译器和扫描的文件夹。

下一步，也许是MakeBoot（笑）？

编译64位程序：

![image-20201109163326280](E:\WorkSpace\blog\md\C++\Image\image-20201109163326280.png)

### 2.最基本的CMakeLists.txt

最基本的CMakeLists.txt指明CMake版本、项目名、生成可执行文件名即可。

```cmake
# 可用于编译该项目的CMake最低版本号
cmake_minimum_required(VERSION 3.10)

# 设置项目名（可通过CMAKE_PROJECT_NAME变量取得）
project(Tutorial)

# 添加可执行文件名，后面追加对应的C++源文件
add_executable(Tutorial tutorial.cpp)
```

CMake的内置命令不区分大小写（如add_executable写成ADD_EXECUTABLE也正确；但CMake的变量都是区分大小写的。

> CMake的CMakeLists.txt并不是源文件，而更类似于批处理文件，其中的project、add_executable不是函数而是命令，参数的分割也不是用","而是使用空格。

之后在项目中新建一个自定义名称文件夹（如build）并进入其中然后运行命令`cmake ..`创建Makefile文件，然后运行`cmake --build`或`make`命令进行编译生成add_executable中指定的二进制文件。

### 3.在C++源文件中访问CMake变量

使用project命令的的VERSION选项设置项目版本号：

```cmake
project(Tutorial VERSION 1.0)
```

CMake可以修改源文件的内容，如同freemarker一般设置一个模板，调用`configure_file(<input> <output>)`命令让CMake填上变量的内容。变量可以使用`@VAR@`或`${VAR}`来引用。

模板如下：

```cmake
#define Test_VERSION_MAJOR @Test_VERSION_MAJOR@
#define Test_VERSION_MINOR @Test_VERSION_MINOR@

#或
#define Test_VERSION_MAJOR ${Test_VERSION_MAJOR}
#define Test_VERSION_MINOR ${Test_VERSION_MINOR}
```

命令如下：

```cmake
configure_file(TestConfig.h.in TestConfig.h)
```

该命令会在输出二进制文件的目录（默认为CMakeLists.txt同级的cmake-build-debug文件夹）下生成一个名为TestConfig.h的头文件，头文件中的变量已被CMake替换为值，内容如下：

```cmake
#define Test_VERSION_MAJOR 1
#define Test_VERSION_MINOR 0
```

该目录并不在默认Include的目录中，可以使用绝对路径引入TestConfig.h获得这些宏的值，代码如下：

```cmake
#include "E:\WorkSpace\CPP\Temp\cmake-build-debug\TestConfig.h"

int main(){
    std::cout << Test_VERSION_MAJOR << std::endl;
}
```

当然更好的做法是将此文件夹纳入此Target（即add_executable生成的一个可执行文件Test.exe或add_library()生成的一个Test.dll）的target_include_directories，取消其路径前缀。可以通过CMake变量PROJECT_BINARY_DIR获得cmake-build-debug目录的绝对路径，CMakeLists.txt中命令如下：

```cmake
target_include_directories(Test PUBLIC "${PROJECT_BINARY_DIR}")
```
注意target_include_directories因为第一个参数是Target名，其应该放在add_executable后执行。

> 除了PROJECT_BINARY_DIR，常用的还有PROJECT_SOURCE_DIR，CMAKE_CURRENT_SOURCE_DIR

代码如下：

```cmake
#include "TestConfig.h"
```

### 4.创建变量

##### ①创建普通变量

在CMake中创建变量使用set命令，和批处理中的变量一样，如果未存在的变量将新建，已存在的变量将赋值，命令如下：

```cmake
set(CMAKE_CXX_STANDARD 11)
```

CMAKE_CXX_STANDARD是一个CMake提供的变量，用于指定C++标准版本（如C++11，C++14，C++17，C++20）。代码如下：

```cmake
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

> 在CMake中CXX代表C++，据说因为考虑部分操作系统不支持+作为文件名的一部分，将+斜过来变成x。

##### ②创建编译条件

使用option命令创建编译条件，可在if-elseif-else中使用。

```cmake
option(USE_MYMATH "Use custom math functions" ON)
if (USE_MYMATH)
    target_include_directories(Test PUBLIC "${PROJECT_BINARY_DIR}" "${PROJECT_SOURCE_DIR}/MathFunctions")
else()
    target_include_directories(Test PUBLIC "${PROJECT_BINARY_DIR}")
endif ()
```

要在C++源文件中使用，和普通的CMake变量一样需要使用configure_file让CMake填上内容，但宏需要使用#cmakedefine，模板如下：

```cmake
#cmakedefine USE_CUSTOM
```

当USE_MYMATH为ON时，此行将会被替换为：

```
#define USE_CUSTOM
```

当USE_MYMATH为OFF时，此行将会被替换为一句注释：

```cmake
/* #undef USE_CUSTOM */
```

在代码中使用：

```cmake
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```

在编译时可以通过命令行传递option的值：

```cmake
cmake .. -DUSE_MYMATH=OFF
```

> 在Windows下CLion中似乎因为不能清除缓存修改option会无效。

##### ③创建列表变量

使用list命令创建和操作列表变量

```cmake
# 向列表EXTERA_LIBS中追加值"MathFunctions"（若不存在此列表将创建）
list(APPEND EXTERA_LIBS MathFunctions)
```

### 5.依赖

C++有依赖的.cpp源文件编译需要三个东西：

①头文件（.h）：提供函数的声明，给程序员和编译器看的（编译器只需要函数的参数和返回值就可以判断调用是否合法，即便对应的函数没有实现也是在链接器中报错）；

②.lib文件：静态链接中含函数体，动态链接中提供函数名与函数体所在的.dll中的位置对应关系，给链接器看的；

③.dll文件：函数体，这里的函数没有声明只有实现，连函数名都只有一个序号，序号与函数名的对应关系在.lib文件中。.dll不参与链接，但是运行的时候需要能够找到（.exe和.dll的依赖在其导入表中）。

> 即便是动态链接.dll，在Windows下也必须静态链接一个.lib，不过这个.lib只是一个符号表。而静态链接的.lib里是有实现的。

例如通过add_library命令创建的一个名为MathFunctions的库

```cmake
add_library(Torch test.cpp)
```

当需要引入该库时，需要使用命令target_link_libraries

```cmake
target_link_libraries(Test PUBLIC Torch)
```

那么这样就可以使用Torch库了吗？当然不是，因为我们还未将Torch库的头文件所在目录加入target_include_directories中，这将导致引入其头文件必须使用绝对路径，这不是麻烦的问题，因为Torch中的代码也会引入自己的头文件，它们使用的是相对路径，会报错。因此通常我们需要了解我们引入的库的结构，将其一一引入取消其路径前缀。

```cmake
target_include_directories(Torch PUBLIC
                          ……
                          "${TORCH_INCLUDE_DIR}"
                          "${TORCH_INCLUDE_DIR}/torch"
                          "${TORCH_INCLUDE_DIR}/torch/csrc/api/include/torch"
                          ……
                          )
```

不过CMake提供了一个简单的机制，让使用库的人不必了解库的结构，而创建库的人需要使用target_include_directories命令指明引入该库时需要加入target_include_directories的文件夹，这种特别的target_include_directories需要使用参数INTERFACE，如下：

```cmake
target_include_directories(MathFunctions
          INTERFACE "${CMAKE_CURRENT_SOURCE_DIR}/include"  "${CMAKE_CURRENT_SOURCE_DIR}/torch" "${CMAKE_CURRENT_SOURCE_DIR}/torch/csrc/api/include/torch"
          )
```

这样target_link_libraries(Test PUBLIC Torch)引入库时不需要再手动引入Torch库的头文件。

### 6.安装

安装是将.h、.dll、.exe拷贝到对应的文件夹。安装的文件夹名通过变量CMAKE_INSTALL_PREFIX或命令参数指定：

```bash
cmake --install . --prefix "/home/myuser/installdir"
```

CMakeLists.txt中

```cmake
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```

## 二.package

类似于pip，CMake将一个第三方库定义为一个package，使用`find_package(<包名>)`命令，CMake将会搜索`path_to_your_cmake/share/cmake-<version>/Modules`下对应的

Find<包名>.cmake文件，该文件通常定义了一些CMake变量，如：

```
<LibaryName>_FOUND
<LibaryName>_INCLUDE_DIR or <LibaryName>_INCLUDES
<LibaryName>_LIBRARY or <LibaryName>_LIBRARIES
```

直接在target_include_directories和target_link_libraries使用这些变量，便可以在不了解第三方库结构的情况下使用它们。

