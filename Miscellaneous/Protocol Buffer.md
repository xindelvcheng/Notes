# Protocol Buffer笔记

Protocol Buffers是Google提出的跨语言跨平台序列化方案。

使用时创建一个.proto数据结构声明，由protocol buffer编译器protoc生成一个类，能够像使用一个普通的类一样使用Getter/Setter读写数据（其拥有.proto中声明的那些字段作为属性），而含有编码为二进制文件和解析二进制文件的能力。

> 这听上去不就是一个跨语言的仅数据类反射方案？

[TOC]

## 一.入门

### 1.编写.proto

使用“Protocol Buffers语言”的message关键字创建一些“类”。

```protobuf
syntax = "proto2";

message Student{
    optional string name = 1;
    optional int32 age = 2;
    optional string address = 3 [ default = "unknown" ];
}
```

与Java/C++相比多了数据类型前的required/optional字段标识符来表示该字段是否必选（必选会引起维护的问题因此避免使用，“必选”的判断应该放在应用层），以及设置默认值使用[default=]的形式，而这里的name=1意为该字段在二进制文件中的唯一标签为1。内建数据类型有bool, int32, float, double, string。使用message声明的Student则是自定义数据类型，此外也可以使用enum声明枚举。

当字段修饰为repeated时，表明该字段为一个数组（数量>=0）：

```protobuf
message Teacher{
    repeated Student students = 1;
}
```

看上去这十分像C++/Java语言，但相比class，protobuf的message是没有继承的。

### 2.编译.proto

对python而言，就是声明.py文件。

##### ①下载protoc编译器

从https://github.com/protocolbuffers/protobuf/releases下载protoc编译器，解压可以得到一个protoc.exe。

##### ②编译并生成.py文件

```python
protoc --python_out=./temp test.proto
```

将在temp文件夹下声明test_pb2.py文件。

> 将这里的--python_out换成--cpp_out或--java_out就可以生成.h&.cc、.java，供其他语言使用。

### 3.在Python中使用

像一个普通类那样使用protoc基于message生成的类：

```python
import test_pb2

student = test_pb2.Student()
student.name = "ZhangSan"
student.age = 20
```

特别地，如果是列表成员（修饰为repeated的字段），如果列表中的元素是message，那么可以调用该成员的add方法生成一个对象加入列表并返回新生成的这个对象的引用。而普通的append方法会将一个拷贝加入列表中。

```python
student1 = test_pb2.Student()
student1.name = "LiSi"
student1.age = 20
teacher.students.append(student1) # 加入的student1是一个拷贝

# 等价于
student2 = teacher.students.add() # 列表中的student2和外面的是同一个
student2.name = "LiSi"
student2.age = 20
```

同时，protoc基于message生成的类自带序列化方法`SerializeToString`、反序列化方法`ParseFromString`：

```python
with open("temp.pb", 'wb') as f:
    f.write(student.SerializeToString())

with open("temp.pb", 'rb') as f:
    deserialized_student = test_pb2.Student()
    deserialized_student.ParseFromString(f.read())
    print("name:{},age:{}".format(deserialized_student.name, deserialized_student.age))
```

> 这里虽然用了str，但序列成的是二进制流，Google仅仅将str用作方便的容器。

注意，不应该继承该类来扩展，只能设计包装类。

