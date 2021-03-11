# Python笔记

[TOC]

## 一.语法

#### 1.Python程序组织方式

##### ①Module

Module，就是**一个Python源文件**。使用import导入一个模块，实际上声明了一个全局变量，名称为`__name__`。

> Python中一切皆对象，函数也是一个对象（重写了`__call__`的可调用对象），函数定义就是在全局变量表中写入一个函数名。类也是一个对象，是它的原类的对象。

##### ②import和from ... import

（1）import

不将导入的模块直接写入当前文件的全局符号表中，实际为执行了模块的`__init__.py`并在当前文件中声明了一个名为`__name__`的全局变量。

一个模块不能被import语句导入多次，如在外部修改了模块，应该重启交互环境或者使用model.reload()。

只能导入包和模块，不能导入模块中的类/函数/变量。

（2）from ... import

将导入的模块部分直接写入当前文件的全局符号表中。可以导入模块中的类/函数/变量（但不推荐这么做）。

（3）示例

```python
In [1]: from json import loads 			 		 //√
    
In [2]: import matplotlib.pyplot as plt			 //√
In [3]: import json.loads						 //×
```

##### ②PYTHONPATH

类似于Java中的classpath，使用`sys.path`查看，默认为以下内容。

```python
['C:\\Program Files\\Python37\\Scripts\\ipython.exe',
 'c:\\program files\\python37\\python37.zip',
 'c:\\program files\\python37\\DLLs',
 'c:\\program files\\python37\\lib',
 'c:\\program files\\python37',
 '',
 'C:\\Users\\26046\\AppData\\Roaming\\Python\\Python37\\site-packages',
 'c:\\program files\\python37\\lib\\site-packages',
 'c:\\program files\\python37\\lib\\site-packages\\win32',
 'c:\\program files\\python37\\lib\\site-packages\\win32\\lib',
 'c:\\program files\\python37\\lib\\site-packages\\Pythonwin',
 'c:\\program files\\python37\\lib\\site-packages\\IPython\\extensions',
 'C:\\Users\\26046\\.ipython']
```

显然，这是一个列表，因此可以使用append追加内容。追加的文件夹路径会成为PYTHONPATH的一部分，并且会缓存模块编译过的二进制文件。

```python
>>>import sys
>>>sys.path.append("D:\\Temp")
>>>import module_test
>>>module_test.fib(10)
```

##### ③pyc文件

当程序从.pyc文件中读取时，其运行速度不会比从.py文件中读取时快多少；.pyc文件唯一快的地方是加载速度。pyc文件可以用于非开源项目交付，它们同样跨平台。

##### ④Package

Python中的模块（module）只是一个文件，大约相当于Java中public class的等级，而和Java中package对应的，是Python中的package（包）。和Java中的package一样，python中的package就是文件夹，且可以互相嵌套。

不同的是，Java中的package确实是普通的文件夹，而Python中的文件夹通常要有一个名为`__init__.py`的文件（即便是空的）才会被视为一个package（在新版本中没有这个限制了，普通的文件夹也会被当作package）。这种行为的本质是不对文件夹做递归判定，以免中间有个如`string`这样的文件夹名，不经意地覆盖了后面地内容（而且这显然非常耗时）。

尽管可以是空的，`__init__.py`，但很少有人会这么做，这个`__init__.py`相当于Android开发中的`AndroidManifest.xml`，不再里面声明的子模块对`from package import *`是不可见的（但`import package.module`依旧有效）——尽管这是一种糟糕的习惯。

##### ⑤Python程序组织方式总结

（1）`from package import submodule`是推荐的语法，`import package`也可以但使用成员时需要加包名，不推荐使用`from package import */function/variables`。

（2）在子模块内导入兄弟模块时，尽管可以使用`.`、`..`这样的前缀来表示相对路径，但推荐使用绝对路径，例如

```python
sound/                          Top-level package
      __init__.py               Initialize the sound package
      effects/                  Subpackage for sound effects
              __init__.py
              echo.py
              surround.py
              reverse.py
              ...
      filters/                  Subpackage for filters
              __init__.py
              equalizer.py
              vocoder.py
              karaoke.py
              ...
```

如果在`vocoder.py`中想要导入`echo.py`，推荐使用`from sound.effects import echo`。不推荐使用`from ..effects import echo`，这种代码很难看。

## 二.Python特有语法和API

### 1.内建函数

##### ①map(function, iterable, ...)

对指定序列做映射生成一个新的序列（返回值是一个生成器而不是列表，因此需要使用list转换）示例代码如下：

```python
array = [1, 2, 3, 4, 5]
index_array = list(map(lambda x: x - 1, array))
```

出现时间比列表生成式早，相当于：

```python
array = [1, 2, 3, 4, 5]
index_array = [x - 1 for x in array]
```

##### ②zip([iterable, ...])

用于序列间的逐元素配对，示例代码如下：

```python
x_data = [1, 2, 3, 4, 5]
y_data = [1, 4, 9, 16, 25]

train_data = list(zip(x_data, y_data))
```

train_data如下：

```python
[(1, 1), (2, 4), (3, 9), (4, 16), (5, 25)]
```

### 2.特有语法

##### ①参数打包和解包

在函数声明时，使用`*args`收集位置参数到元组中，使用`**args`收集`关键字参数到字典中，示例代码如下：

```python
def func(arg, *args, **kwargs):
    arg1 = arg
    arg2 = args[0]
    arg3 = args[1]
    arg4 = kwargs["keyname"]
```

调用

```python
func(10, 20, 30, keyname=40)
```

在函数调用时，使用`*variable`可以将元组或列表解包成位置参数，使用`**`可以将字典解包成关键字参数，上面的调用等价于：

```python
a = 10
li = [20, 30]
d = {"keyname": 40}
func(a, *li, **d)
```

## 二.代码片段

##### 1.添加路径到环境变量

```python
base_dir = os.path.dirname(__file__)
sys.path.append(base_dir)
```

##### 2.Socket通信

```python
import socket

server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(("127.0.0.1", 6666))
server_socket.listen(128)
while True:
    connection_socket, addr = server_socket.accept()
    connection_socket.send(b'Message from python socket')
```



