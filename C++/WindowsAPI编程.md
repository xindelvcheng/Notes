# WindowsAPI编程

[TOC]

一.简介

WindowsAPI提供操作系统接口，它是清一色的C风格接口。处于习惯，也称为Win32接口，使用的dll总是会被命名为xxx32.dll，但其在64位环境中包含64位二进制文件（即便名字是32）

它有以下几类：

1. 基础服务：kernel32.dll，包含内核函数
2. 图形设备接口：dpi32.dll，包含管理设置的函数
3. 图形化用户界面：user32.dll，GUI函数
4. 通用控件库：comctl.dll，控件库
5. 通用对话框链接库：comdlg32.dll
6. Windows Shell：shell32.dll
7. 网络服务：ws2_32.dll