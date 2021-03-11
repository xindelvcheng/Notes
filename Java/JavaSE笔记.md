# JavaSE笔记

[TOC]

javaSE就是Java的基础内容。

Java源代码文本文件被JDK编译为`.class`二进制字节码文件，然后被`ClassLoader`加载进内存并创建对应的`Class`对象，jvm通过这个Class对象就掌握了源代码中的信息，可以新建对应的类实例、调用它里面的方法。

## 一.语法

#### 1.Lambda表达式

配合函数式接口使用。避免为了一个方法多写一个实现类的冗余代码。

如果在Lambda表达式中使用了外部的变量，实际上就是一个闭包（词法闭包，Lexical Closure，在一个函数中嵌套定义函数，后者可以访问前者的变量）。

```java
new Thread(()->{
    System.out.println("Hello!");
}).start();
```

#### 2.注解

annotation，使用类似定义接口的形式，定义一些抽象方法。是给编译器和解析程序用的，常用于框架中结合反射读取配置信息。

`getAnnotation(Class)`实际上是在内存中生成了一个该注解（接口）的实现类，其各方法的返回值为使用注解时传入的参数（属性）。

## 二.常用API

#### 1.集合

##### （1）Array

```java
Arrays.copyOfRange(int[] original, int from, int to) 
```

##### （2）Map

```java
put(Object key, V Value)

getOrDefault(Object key, V defaultValue) 
```

#### 2.时间

##### （1）LocalDateTime.now()

##### （2）LocalDateTime.now().toLocalDate()

##### （3）程序计时

```java
double start = System.currentTimeMillis();

long res = fib_recursion(44);

double end = System.currentTimeMillis();
System.out.format("Thread Name:%s,Result:%d,Timeout:%fs", Thread.currentThread().getName(),res,(end-start)/1000);
```

#### 3.IO

##### （1）文件写入

​	写入文本文件

```java
try {
    BufferedWriter bw = new BufferedWriter(new FileWriter("D:log.txt"));
    bw.write("日志");
    bw.newLine();
    bw.write(LocalDateTime.now().toLocalDate().toString());
    bw.close();
} catch (Exception e) {
    e.printStackTrace();
}
```

## 三.进阶

### 1.多线程

因为Web容器自身是多进程/多线程的，一般不需要Java程序员写多线程的代码。

```java
public static void main(String[] args) throws InterruptedException {
    for (int i = 0; i < 10; i++) {
        new Thread(() -> {
            long start = System.currentTimeMillis();
            res += fib(41);
            long end = System.currentTimeMillis();
            System.out.println(end-start);
        }).start();
    }
    Thread.sleep(1000);
    System.out.println(res);
}
```

注意，如果要使用多线程加速运算，不能用join。

Thread.join把指定的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。

比如在线程B中调用了线程A的join()方法，直到线程A执行完毕后，才会继续执行线程B。

### 2.项目管理工具

Maven和Gradle是最常见的两种项目管理工具，它们主要有两个功能：

①定义项目的通用目录结构

一个功能模块分为以下四个文件夹

—src

——main.java					  存放项目源代码

——main.resources			存放源码配置文件

——test								 存放单元测试代码

——test.resources			   存放单元测试代码配置文件

②依赖管理

通过Maven（pom.xml）和Gradle（Groovy编程语言）统一管理jar包。

它们都是纯jar编写的本质都是都是一个批处理文件和依赖/启动的jar包。

##### （1）Maven

在pom.xml中写好配置文件，使用`mvn clean package`（编译）、`mvn clean install`（安装到本地）、`mvn clean deploy`（发布到远程仓库）。

一个最简单的pom.xml如下。

使用\<dependency>加入需要的依赖。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.2.7.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
   </parent>
   <groupId>com.xindelvcheng</groupId>
   <artifactId>demo</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>demo</name>
   <description>Demo project for Spring Boot</description>

   <properties>
      <java.version>1.8</java.version>
   </properties>

   <dependencies>
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
      </dependency>

      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-test</artifactId>
         <scope>test</scope>
         <exclusions>
            <exclusion>
               <groupId>org.junit.vintage</groupId>
               <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
         </exclusions>
      </dependency>
   </dependencies>

   <build>
      <plugins>
         <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>
</project>
```

##### （2）Gradle

Gradle使用Grovvy**编程语言**在build.gradle中写构建脚本（这是与Maven的不同之处，build.gradle并不是要给配置文件，而是一个运行在jvm上的脚本）。

Grovvy是一种类似Python的脚本语言，但**完全兼容java的语法**。

Gradle构建项目的过程就是执行一遍build.gradle

但之所以build.gradle看起来不像是编程语言，是因为Gradle为了简化编写而添加的一些语法糖（亦或称为约定），它们大多是符合直觉但是不符合语法的，且目的大多是在不引起歧义的情况下省略小括号。

①在不引起歧义的情况下可以省略小括号，如`println("Hello!")`可以简化成`println "Hello!"`，

②闭包（Grovvy中其实就是方法指针）只要用大括号括起来可以直接当变量使用，不用一定要`def increase = {e->++e}`然后可以`increase(10)`。

③闭包如果不写return，默认返回最后一行的返回值。

④闭包如果是最后一个参数，可以不写在小括号里

⑤别问，问就是语法糖，毕竟这是脚本语言，不是编译语言，所有的语句都是反射动态调用的。

根据以上三条规则

```groovy
def f(int i,Closure c){
     println(c(i))
}
def c = {e->return ++e}
f(10,c)
```

简化成了

```groovy
def f(int i,Closure c){
    println c(i)
}

f 10,{e->++e;}
```

或

```groovy
def f(int i,int j,Closure c){
     println(c(i)+c(j))
}

f(10,20) {e->return ++e}
```





##### （3）Gradle相比Maven

##### 优化了运行的模式，它有一个常驻后台的进程负责处理命令而不是像Maven那样每执行一次命令就启动一次jvm。