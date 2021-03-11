# GitHub 教程

[TOC]

## 一.设置SSH密匙

```bash
$ ssh-keygen -t rsa
```

三次回车之后复制`%HOME%/.ssh/id_rsa.pub`中的内容，在Github中settings$\rightarrow$SSH and GPG keys$\rightarrow$New SSH key。

## 二.使用开源库

Code标签：源码

Code/Releases：二进制文件

Wiki：文档

### **三.Quick setup** — if you’ve done this kind of thing before

[ Set up in Desktop](https://desktop.github.com/)

**or**

HTTPSSSH



Get started by [creating a new file](https://github.com/xindelvcheng/Book-DeepLearning/new/main) or [uploading an existing file](https://github.com/xindelvcheng/Book-DeepLearning/upload). We recommend every repository include a [README](https://github.com/xindelvcheng/Book-DeepLearning/new/main?readme=1), [LICENSE](https://github.com/xindelvcheng/Book-DeepLearning/new/main?filename=LICENSE.md), and [.gitignore](https://github.com/xindelvcheng/Book-DeepLearning/new/main?filename=.gitignore).

### …or create a new repository on the command line



```
echo "# Book-DeepLearning" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:xindelvcheng/Book-DeepLearning.git
git push -u origin main
                
```

### …or push an existing repository from the command line



```
git remote add origin git@github.com:xindelvcheng/Book-DeepLearning.git
git branch -M main
git push -u origin main
```

### …or import code from another repository

You can initialize this repository with code from a Subversion, Mercurial, or TFS project.

[Import code](https://github.com/xindelvcheng/Book-DeepLearning/import)

### 同步fork仓库

并没有专门的命令用于同步fork的仓库，原理是将fork加入remote中，然后从原仓库拉到本地，再推到自己的仓库

```bash
$ git remote add upsteam git@github.com:EpicGames/UnrealEngine.git
$ git pull upstream #或git pull upstream release
$ git push origin
```

