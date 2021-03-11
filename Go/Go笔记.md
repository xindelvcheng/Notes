# Go笔记

[TOC]

## 一.入门

Go语言来源于C语言的创造者，因此可以看作C语言真正意义上的升级版。

#### 1.Hello World

```go
package "main"//①

import "fmt"
import "math"

func main() {//②
	fmt.Println(math.pi)//③
}
```

①包

package，go语言程序的组织形式

②主函数

main

③外部变量

Go规定凡是首字母大写的变量和函数都是public的，凡是小写的就是private的

#### 2.变量

有两种语法声明并赋值变量，使用和C/C++相同。

①普通定义

变量类型在变量名后面，比较繁琐，一般用于全局变量定义

```go
var i int = 10
```

②快速定义，缺点是不能在函数外使用，一般用于定义局部变量

```go
i := 0
```

#### 3.函数

Go的函数定义比较特别，且可以返回任意数量的变量

```go
func add(x int, y int) int {
	return x + y
}

func swap(x, y string) (string, string) {
	return y, x
}
```

Go 的返回值可以被命名，并且像变量那样使用。

```go
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

#### 4.for

Go 只有一种循环结构——`for` 循环。但`for`可以省略初始化语句和自增语句，当作`while`使用。如果连条件语句都省略了，就变成了死循环。

基本的 `for` 循环除了没有了 `( )` 之外（甚至强制不能使用它们），看起来跟 C 或者 Java 中做的一样，而 `{ }` 是必须的。

```go
package main

import "fmt"

func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```

#### 5.if

`if` 语句除了没有了 `( )` 之外（甚至强制不能使用它们），看起来跟 C 或者 Java 中的一样，而 `{ }` 是必须的。和C/C++中不一样的是，跟 `for` 一样，`if` 语句可以在条件之前执行一个简单的语句。

```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// 这里开始就不能使用 v 了
	return lim
}
```

#### 6.指针

Go中也有指针，使用方式和C/C++中相同的，但不能进行运算。

```go
func main() {
	i := 0
	p := &i
	*p = 1
	fmt.Println(i)
}
```

#### 7.类和对象

Go中使用`type`定义结构体`Struct`（而非`class`），没有对象的概念。结构体中只能定义属性，方法必须以扩展的方式定义在外面。

