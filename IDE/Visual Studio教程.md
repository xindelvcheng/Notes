# Visual Studio教程

[TOC]

## 一.安装

官网：https://visualstudio.microsoft.com/zh-hans/vs/

## 二.快捷键

Ctrl+K，O：切换头/源文件

Ctrl+Alt+Left/Right：跳转到上一次编辑处

Ctrl+B：跳转到定义

## 三.其他

#### 1.快捷键插件
​	vsixgallery：http://vsixgallery.com/

##### （1）Vim

​	主流IDE均有Vim插件。Visual Studio中插件为[VsVim](https://marketplace.visualstudio.com/items?itemName=JaredParMSFT.VsVim)，GitHub：https://github.com/VsVim/VsVim。

​	VsVim会扫描多个位置寻找配置文件，建议为`%HOME%/.vimrc`

```
inoremap jj <esc>
```

​	`inoremap`命令用于映射按键。i代表是在插入模式（insert）下有效nore表示不递归no recursion，例如：`inoremap Y y`和`inoremap y Y`并不会出现无限循环，map映射。

##### （2）Hot Keys

​	安装了之后会提示添加了3个新的Shortcut，可以通过Tools$\rightarrow$Keyboard Shortcuts$\rightarrow$Load Shortcuts启用。

#### 2..Net开发环境

##### （1）设置为启动项目

​	所有的项目都会编译，但只有设置为启动项目的才会运行。

##### （2）Nuget

​	右键项目打开管理Nuget包。

### 3.添加文件夹

选中显示所有文件

<img src="E:\WorkSpace\blog\md\IDE\Image\image-20201016195123718.png" alt="image-20201016195123718" style="zoom:50%;" />