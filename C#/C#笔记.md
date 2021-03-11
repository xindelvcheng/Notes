# C# 笔记

[TOC]

## 一.文件系统

文件是由操作系统管理的，在C#获取关于驱动器（Windows上一般指硬盘）、文件、文件夹的信息，可以调用DriveInfo、DirectoryInfo 、FileInfo 的接收一个路径参数的构造函数获得一个对应的Info对象，其中存储了文件的各种属性。也可以使用`System.IO.Directory`和 `System.IO.File`类提供的相关静态方法获得这些对象。