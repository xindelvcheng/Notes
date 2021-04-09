# Python常用库

[TOC]

常用库都可以通过`pip install`的方式安装，Windows下需要以管理员方式运行CMD，Linux下需要添加pip为pip3的别名。对只有源码的库，使用`python setup.py install`安装。安装库后应重启终端，否则可能会有一些奇奇怪怪的问题。

获取任意库的安装位置可以导出库后查看其`__file__`属性。	

## 一.标准库

#### 1.re

用`()`包裹需要提取的字符串

```python
re.findall('<html>(.*)</html>','<div class="content">sdf</div><div>sdf</div>')
```

##### ①匹配手机号

```
re.findall('1\d{10}',text)
```

##### ②匹配邮箱

```python
re.findall('\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}',text)
```

##### ③匹配中文

```python
re.findall('[\u4e00-\u9fa5]+',text)
```

#### 2.pylint

用于格式检查和纠错。

## 二.数据库

#### 1.SQLite

SQLite是一款基于文件的、无需安装的轻量级的数据库，内置于Python中。

##### ①创建/连接数据库

```python
import sqlite3
conn = sqlite3.connect('test.db')
```

**注**：当使用“`:memory`作为数据库名时，会在Ram上创建一个数据库。

##### ②创建表

```python
c = conn.cursor()
c.execute('''CREATE TABLE GeographyInfo
      (ID INT PRIMARY KEY     NOT NULL,
       NAME           TEXT    NOT NULL,
       X		      REAL    NOT NULL,
       Y		      REAL    NOT NULL,
       LEVEL          INT	  Not NULL,
       DETAIL         TEXT);''')
conn.commit()
conn.close()
```

执行增删改查都可以通过`c.execute(sql)`的方式进行。cursor中也存储了查找到的数据。

##### ③增删改查

增

```python
conn = sqlite3.connect('test.db')
c = conn.cursor()
c.execute("INSERT INTO GeographyInfo (ID,NAME,X,Y,DETAIL) \
      VALUES (1, 'Hefei', 117.282699,31.866942, 'a growing city' )")
```

删

```python
c.execute("DELETE from GeographyInfo where ID=1;")
```

改

```python
c.execute("UPDATE GeographyInfo set DETAIL = 'The capital of Anhui Province' where ID=1")
```

查

```python
conn = sqlite3.connect('test.db')
c = conn.cursor()
cursor = c.execute("SELECT *  from GeographyInfo")
//(1)
for row in cursor:
    print(row)
//(2)
results = cursor.fetchall()
//(3)√
result = cursor.fetchone()
```

## 二.NLP

#### 1.PyPinyin

①安装

```bash
$ pip install pypinyin
```

②使用

```python
>>> from pypinyin import pinyin, lazy_pinyin, Style
>>> pinyin('中心')
[['zhōng'], ['xīn']]
>>> pinyin('中心', heteronym=True)  # 启用多音字模式
[['zhōng', 'zhòng'], ['xīn']]
>>> pinyin('中心', style=Style.FIRST_LETTER)  # 设置拼音风格
[['z'], ['x']]
>>> pinyin('中心', style=Style.TONE2, heteronym=True)
[['zho1ng', 'zho4ng'], ['xi1n']]
>>> pinyin('中心', style=Style.TONE3, heteronym=True)
[['zhong1', 'zhong4'], ['xin1']]
>>> pinyin('中心', style=Style.BOPOMOFO)  # 注音风格
[['ㄓㄨㄥ'], ['ㄒㄧㄣ']]
>>> lazy_pinyin('中心')  # 不考虑多音字的情况
['zhong', 'xin']
```

### 2.google_trans_new

Google翻译。

> google_trans因为Google翻译API改动已经无法使用了。

##### ①安装

```bash
$ pip install google_trans_new
```

##### ②使用

```python
from google_trans_new import google_translator

translator = google_translator(timeout=10)

translated_text = translator.translate(["你好啊！"], lang_src='auto', lang_tgt='en')
```

## 三.其他

#### 1.mistune

[mistune](https://github.com/lepture/mistune)是一个Python编写的开源Markdown渲染引擎（正在更新2.0，但默认安装0.8.4）

```bash
$ pip install mistune
```

