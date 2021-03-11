# C++笔记

[TOC]

## 一.Socket通信

以下代码适用于Linux环境（Windows中WSL即可）。

### 1.Socket简介

socket编程是一种网络IO编程，其中“IP：端口号”起到门牌号的作用。

> 通常一台电脑只有一块网卡，也只有一个IP地址，用于在网络上定位；电脑上正在运行的程序每个则可以占用多个端口，用于在进程中定位。IP:端口在网络环境中唯一标识一个进程。

当套接字创建成功时，得到是一个文件描述符（伪文件），操作的是一块内核缓冲区（读缓冲区+写缓冲区）。当传输数据时，先写入本机的写缓冲区，传输到对方的读缓冲区。即Socket操作被实现为文件读写，而写缓冲区和对方读缓冲区的数据传输是自动的 （只要写缓冲区有内容就会被发送）。因为文件读写的Read、Write的阻塞与否与文件的性质有关，因此Socket的是否阻塞也是文件描述符决定的（默认阻塞）。

> 内核使用文件描述符（fd）来访问文件，它是一个非负整数。

### 2.入门

在Linux中进行Socket编程需引入头文件：

```c++
#include <unistd.h>
#include <sys/socket.h>
#include <arpa/inet.h>
```

##### (1)创建Socket

使用函数`int socket (int __domain, int __type, int __protocol)`：

__domain：协议，可选AF_INET(IPv4)和AF_INET6(IPv6)

 __type：传输方式，可选SOCK_STREAM（TCP流式传输），SOCK_DGRAM（Datagram缩写，UDP报式传输）

__protocol：填0，表示使用对应传输方式的默认协议（这个参数用于选择特定类型Socket使用的协议，但通常一种类型只有一个协议支持）

此函数成功则返回新创建的socket的文件描述符，失败返回-1。

示例：

```c++
int lfd = socket(AF_INET, SOCK_STREAM, 0);
```

> C语言既不支持默认参数也不支持函数重载，所以会有一些看起来没必要的参数设置。

##### (2)服务器绑定IP端口号

使用函数`int bind (int __fd, const struct sockaddr *__addr, socklen_t __len)`

__fd：socket函数返回的文件描述符

__addr：sockaddr_in 指针类型的描述结构体（用于指定IP和端口号）

\__len：__addr结构体的长度（因为传入的指针，函数内部无法得知该结构体的大小）

> ①客户端可以不调用该函数，操作系统会自动分配一个（IP和）端口号。
>
> ②因为历史原因，该函数要求的是sockaddr*类型的参数，但该结构体已经废弃，需要创建sockaddr_in类型结构体并在传入时强转。

示例：

```c++
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(6666); /*16 bit数据 host（小端） to network（大端）*/
server_addr.sin_addr.s_addr = htonl(INADDR_ANY) /*有效IP*/;
bind(lfd, (sockaddr *)&server_addr, sizeof(server_addr));
```

##### (3)服务器指定同时建立连接的上限数（排队建立三次握手）

使用函数`int listen (int __fd, int __n)`

__fd：socket函数返回的文件描述符

__n

示例：

```c++
listen(lfd, 128);
```

##### (4)服务器等待连接

调用函数`int accept (int __fd, struct sockaddr *__addr, socklen_t *__restrict __addr_len)`

__fd：socket函数返回的文件描述符

__addr：sockaddr 指针类型的描述结构体（用于传出连接的客户端的IP和端口号）

__addr_len：传入传出参数，调用时传入用于接受结果的sockaddr 结构体的大小，之后在函数内部会改写该值符合返回结果的大小

调用该函数后程序将会阻塞，直到有客户端连接，该函数才执行完成并返回一个新的Socket的文件描述符，使用该文件描述符可以与客户端通信。

示例：

```c++
sockaddr_in client_addr;
socklen_t addr_len = sizeof(client_addr);
int cfd = accept(lfd, (sockaddr *)&client_addr, &addr_len);
```

##### (5)连接服务器

使用函数`int connect (int __fd, const struct sockaddr * __addr, socklen_t __len)`

__fd：socket函数返回的文件描述符

__addr：sockaddr 指针类型的描述结构体（用于指定IP和端口号）

\__len：__addr结构体的长度（因为传入的指针，函数内部无法得知该结构体的大小）

可使用`nc 127.0.0.1 6666`模拟客户端连接。

##### (6)客户端通过Socket中发送数据

使用文件IO函数`ssize_t write (int __fd, const void *__buf, size_t __n)`

__fd：socket函数返回的文件描述符

__buf：缓冲区指针（即定义的一个数组）

__nbytes：发送字节数

示例：

```c++
char buffer[4096];
write(cfd, buffer, n);
```

##### (7)服务器从Socket中获取数据.

使用文件IO函数`read (int __fd, void *__buf, size_t __nbytes)`

__fd：socket函数返回的文件描述符

void *：缓冲区指针（即定义的一个数组）

__nbytes：读取字节数

```c++
char buffer[4096];
int n = read(cfd, buffer, sizeof(buffer));
```

测试Linux命令`nc 127.0.0.1 6666`

