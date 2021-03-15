# Mysql教程

**注意：可以使用兼容的[MariaDB](https://mariadb.com/)；**

**注意：Mysql不需要单独下载Navicat之类的客户端，用自带的HeidSql或DataGrip都可**

## 一.入门

[TOC]

#### 1.安装

MariaDB的安装非常简单。

#### 2.Idea中关联Mysql数据库

不需要指定数据库名。

可能会报错`关联mysql失败_Server returns invalid timezone. Go to 'Advanced' tab and set 'serverTimezon'`，这是时区错误，要在mysql中运行`set global time_zone='+8:00';`

![image-20200301122502756](Mysql.assets/image-20200301122502756.png)

**注意，MySQL 中 ``Schema` `等价于 数据库，Idea中使用的也是Schemas而非databases。**

#### 3.数据库管理

##### （1）创建数据库

```sql
CREATE DATABASE IF NOT EXISTS <数据库名> DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```

##### （2）删除数据库

```sql
drop database <数据库名>;
```

##### （3）列出所有数据库

```sql
show databases;
```

##### （4）选择数据库

​	各个数据库之间表名是可以重复的

```sql
use <数据库名>;
```

##### （5）显示所有表

```sql
show tables;
```

##### （6）创建表

```sql
CREATE TABLE Person(
   id INT NOT NULL AUTO_INCREMENT,
   name VARCHAR(100) NOT NULL,
   phone_number VARCHAR(20) NOT NULL,
   age INT NOT NULL,
   submission_date DATE NOT NULL,
   PRIMARY KEY ( id )
)ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

**注意：**MySQL命令终止符为分号 **;** 。

**注意：** **->** 是换行符标识，不要复制。

**注意：**创建 MySql 的表时，表名和字段名外面的符号 **`** 不是单引号，而是英文输入法状态下的反单引号，也就是键盘左上角 **esc** 按键下面的那一个 **~** 按键，坑惨了。

反引号是为了区分 MySql 关键字与普通字符而引入的符号，一般的，表名与字段名都使用反引号。

##### 注意：MySQL 字段属性应该尽量设置为 NOT NULL

除非你有一个很特别的原因去使用 NULL 值，你应该总是让你的字段保持 NOT NULL。

##### （7）从文件导入数据

文件需要使用`\t`和`\N`（表示空值）格式化成类csv。

```sql
LOAD DATA LOCAL INFILE 'dump.txt' INTO TABLE mytbl
```

#### 4.增删改查

##### （1）增

```sql
INSERT INTO table_name  (field1, field2,...fieldN)  VALUES  (【valueA1】,【valueA2】,...【valueAN】),(【valueB1】,【valueB2】,...【valueBN】),(【valueC1】,【valueC2】,...【valueCN】)......;

INSERT INTO Person (NAME,phone_number,age,submission_date) VALUES("张三","17311111111",20,'2020-01-01 00:00:01.63');
```

##### （2）删

```sql
DELETE FROM 【table_name】 【where Clause】;
```

##### （3）改

```sql
UPDATE 【table_name】 SET 【col_name】=【value】 【where Clause】;
```

##### （4）查

```sql
select * from 【table_name】;
select 【column_name】 from 【table_name】;
SELECT URL_Part_domain_extract FROM htmls WHERE html_text REGEXP '((([A-Za-z]{3,9}:(?:\/\/)?)(?:[-;:&=\+\$,\w]+@)?[A-Za-z0-9.-]+(:[0-9]+)?|(?:ww••w.|[-;:&=\+\$,\w]+@)[A-Za-z0-9.-]+)((?:\/[\+~%\/.\w-_]*)?\??(?:[-\+=&;%@.\w_]*)#?••(?:[\w]*))?)';
```

**注意**：

更新参数时不要忘记加条件指定哪一行！否则会覆盖所有行的数据！

**where：**数据库中常用的是where关键字，用于在初始表中筛选查询。它是一个约束声明，用于约束数据，在返回结果集之前起作用。

**group by:**对select查询出来的结果集按照某个字段或者表达式进行分组，获得一组组的集合，然后从每组中取出一个指定字段或者表达式的值。

**having：**用于对where和group by查询出来的分组经行过滤，查出满足条件的分组结果。它是一个过滤声明，是在查询返回结果集以后对查询结果进行的过滤操作。

**执行顺序**

select –>where –> group by–> having–>order by

## 二.索引

索引参考B+树（一种m叉排序树）,能调高检索的速度，但降低更新的速度，并占用一定的磁盘空间。

#### 1.建立索引

```sql
CREATE INDEX indexName ON mytable(username(length)); 
```