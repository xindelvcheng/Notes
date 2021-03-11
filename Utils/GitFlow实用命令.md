# GitFlow实用命令

[TOC]

Git Flow是使用Git进行项目开发的范式。

#### 1.创建develop分支

```
git branch develop
git push -u origin develop    
```

#### 2.开始新Feature开发

```
# 切换到dev分支上,接着跟远程的origin地址上的develop分支关联起来
git checkout -b some-feature develop
git push -u origin some-feature    

# 做一些改动    
git status
git add some-file
git commit    
```

#### 3.完成Feature

```
git pull origin develop
git checkout develop
git merge --no-ff some-feature
git push origin develop

git branch -d some-feature

# If you pushed branch to origin:
git push origin --delete some-feature    
```

#### 4.开始Relase

```
git checkout -b release-0.1.0 develop

# Optional: Bump version number, commit
# Prepare release, commit
```

#### 5.完成Release

```
git checkout master
git merge --no-ff release-0.1.0
git push

git checkout develop
git merge --no-ff release-0.1.0
git push

git branch -d release-0.1.0

# If you pushed branch to origin:
git push origin --delete release-0.1.0   


git tag -a v0.1.0 master
git push --tags
```

#### 6.开始Hotfix

```
git checkout -b hotfix-0.1.1 master    
```

#### 7.完成Hotfix

```Git Flow代码示例
git checkout master
git merge --no-ff hotfix-0.1.1
git push


git checkout develop
git merge --no-ff hotfix-0.1.1
git push

git branch -d hotfix-0.1.1

git tag -a v0.1.1 master
git push --tags
```