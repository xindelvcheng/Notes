#  Neo4j

[TOC]

Neo4j是开源Java图数据库，常用于NLP领域。图数据库有Node、Properties 、Relationships的概念，其中Node+Properties 相当于关系型数据库中的一条记录，Node是记录名，Properties 是存储数据的键值对，多了的是节点间的关系，Relationships本身也像一条记录，有自己的Properties 。这样看，一个图数据库相当于一个存储节点一个存储关系的两个关系型数据库。

例如Node：张三，Node的Properties ：姓名、年龄，Relationships：张三 认识 李四，Relationships的Properties ：在公司中。

## 一.入门

### 1.安装

对Ubuntu

```bash
$ wget -O - https://debian.neo4j.org/neotechnology.gpg.key | sudo apt-key add -
$ echo 'deb https://debian.neo4j.org/repo stable/' | sudo tee /etc/apt/sources.list.d/neo4j.list
$ sudo apt-get update
$ sudo apt-get install neo4j
```

### 2.启动

```bash
$ neo4j start

...
Started neo4j (pid 7093). It is available at http://localhost:7474/
...
```

可能需要sudo。之后可以使用在web后台管理，进入后自动执行命令“:server connect”，默认账号密码都是neo4j，连接后设置密码，自动执行:play start，显示使用向导。

### 3.使用

使用Neo4j数据库需要使用Cypher语言而不能使用SQL，在Neo4j顶端输入条中输出命令执行。

##### ①增

使用create命令创建节点。

```cypher
create(zhangsan:Person{id:1,name:"ZhangSan",age:20})
```

这里的zhangsan:Person中zhangsan是节点名，Person是组名，与编程语言中的类相似，但一个节点可以有多个组名。

##### ②查

使用match命令查询数据

```cypher
match (person:Person) return person.name,person.age
```

使用where条件查询

```cypher
match(person:Person) where person.age > 18 return person.name
```

使用order by排序（默认升序）

```cypher
match (person:Person) return person.name,person.age order by person.id
```

使用order by排序（降序）

```cypher
match (person:Person) return person.name,person.age order by person.id desc
```

函数

```cypher
match (person:Person) return toUpper(person.name),person.age
```

聚合

```cypher
match (person:Person) return avg(person.age)
```

##### ③find_or_create

使用merge命令，如果节点存在则返回节点，节点不存在则创建节点

```cypher
merge(zhangsan:Person{id:1,name:"ZhangSan",age:20}) return zhangsan.name,zhangsan.age
merge(lisi:Person{id:2,name:"LiSi",age:25}) 
```

##### ④创建关系

使用create命令，只能创建有方向性的关系。

```cypher
create (zhangsan:Person) -[relationship1:Known]-> (lisi:Person)
```

使用merge命令，既可以创建有方向性的关系也可以创建无方向性的关系。

```cypher
merge (zhangsan:Person) -[relationship2:Colleague]- (lisi:Person)
```

##### 索引

使用命令create index on 命令在一个组的某个属性上索引，加快检索速度。

```cypher
create index on:Person(id)
```

使用drop index on 删除索引

```cypher
drop index on:Person(id)
```

##### ⑥删

使用delete命令删除节点或关系，需与match配合使用。

```cypher
match (person1:Person)-[r]-(person2:Person) delete person1,person2,r
```

> 如果删节点，也要将对应的边删除。

### 4.在Python中使用

##### ①安装驱动

```bash
$ pip install neo4j-driver
```

##### ②导包

```python
from neo4j import GraphDatabase
```

③创建driver连接数据库
```python
driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "admin"))
```

④创建session执行cypher语句

```python
with driver.session() as session:
    cypher = 'create(zhangsan:Person{id:1,name:"ZhangSan",age:20}) return zhangsan.name'
	result = session.run(cypher)
    print(list(result)[0])
```

⑤事务

```python
from neo4j import GraphDatabase

def add_pair(tx, name1, name2):
    cypher = 'merge(person1:Person{id:1,name:$name1})' \
             'merge(person2:Person{id:2,name:$name2})' \
             'merge(person1)-[relationship1:Known]-(person2)'
    tx.run(cypher, name1=name1, name2=name2)

driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "admin"))
with driver.session() as session:
    session.write_transaction(add_pair, "zhangsan", "lisi")
```









