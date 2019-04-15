title: "MySQL架构"
date: 2019-04-15 22:53:50 +0800
update: 2019-04-15 22:53:50 +0800
author: me
cover: "-/images/mysql-jiagou.png"
tags:
    - MySQL
preview: MySQL服务器由SQL层和存储引擎层构成，SQL层主要功能包括处理客户端请求、权限判断、SQL解析和查询缓存处理等，存储引擎层完成底层数据库的数据存储操作。

---

## 一、MySQL架构

MySQL服务器由SQL层和存储引擎层构成，SQL层主要功能包括处理客户端请求、权限判断、SQL解析和查询缓存处理等，存储引擎层完成底层数据库的数据存储操作。

### 1.1 SQL层

+ 客户端通过连接/线程处理层来连接MySQL数据库，连接/线程处理层主要用来处理客户端请求、身份验证和数据库安全性验证等;
+ 查询缓存和查询分析器是SQL层的核心部分，其中主要涉及查询的解析、优化、缓存以及所有内置函数、存储过程、触发器、视图等功能;
+ 优化器主要负责存储和获取所有存储在MySQL中的数据。

### 1.2 存储引擎层

常见的存储引擎层有MyISAM、InnoDB、MEMORY、MERGE、NDB。

查看当前默认存储引擎，使用如下命令:

```
show variables like 'table-type';
show variables like 'have%';
```

修改表的存储引擎的命令为:

```
alter table tablename engine=innodb;
```

创建表时也可以添加默认的存储引擎:

```
create table tablename(
)engine=MyISAM default charset=gbk;
```

## 二、MySQL各种存储引擎的特性

尽管MySQL数据库支持种类繁多的数据存储引擎，但是数据存储不管是使用哪种存储引擎，所有的存储数据都被记录到.frm文件中，该文件记录了存储的数据以及表的一些属性值;不管是使用哪种数据存储引擎，都使用了高速缓存，数据库读取.frm文件信息后会将表的信息缓存起来，提高了服务器下次读取数据的速度。

### 2.1 MyISAM

Linux平台下，MySQL默认的存储引擎是MyISAM，该存储引擎不支持事务和主键，但是对数据的存储和批量查询速度比较快。因此在实际应用中主要对以查询和增加记录为主的应用采用MyISAM存储引擎。MyISAM存储引擎只缓存索引，对数据文件采用操作系统缓存，如果索引数据超过系统所分配的缓存空间时也会采用操作系统来缓存索引。

#### 2.1.1 MyISAM文件格式

+ frm文件:存储表的定义数据;
+ ‌MYD文件:存放表具体记录数据;
+ ‌MYI文件:存储索引。

#### 2.1.2 MyISAM文件修复

MyISAM类型的表在数据存储过程中可能会发生错误，可以通过myisamchk工具修复损坏的表。

#### 2.1.3 MyISAM表的存储格式

该类型的表支持三种不同类型格式的表，分别为

+ 静态(固定长度)表
+ 动态可变长度表
+ 压缩表

其中静态和动态数据存储根据起表中数据列的类型自动选择，静态表是默认的存储格式，压缩表只能通过myisampack工具创建。

静态表中的字段都是固定非变长的字段，这样每个记录都是固定长度的，这种存储方式的优势在于存储速度非常快、容易缓存，表发生缺损后容易恢复，缺点是静态表所占用的空间往往比动态表多。

myisamchk文件修复，该命令格式为

```
myisam [options] tables[.MYI]
```

示例为

```
[root@localhost books]# ls
books.frm books.MYD books.MYI db.opt
[root@localhost books]# myisamchk -e books.MYI
```

动态表支持动态可变长度，字符型的列长是可变的，除了小于4个字符的。动态可变长字符通常比静态固定格式需要更少的存储空间，由于采用动态可变长度存储，所以出错的时候也比静态格式恢复更加困难。
压缩表需要的磁盘存储空间最小，可以通过myisampack工具来压缩MyISAM表，每行单独压缩，每列的压缩也不一样。

### 2.2 InnoDB

该存储引擎写的处理相对于MyISAM效率低一些，其牺牲了存储和查询的效率，但是支持事务安全、支持自动增长列以及外键约束。

### 2.3 MEMORY

该存储引擎通过使用内存中的内容来创建表，每个MEMORY表实际上和一个磁盘文件关联起来，文件名采用.frm的形式。该类型的表访问速度非常快，因为数据来源于内存空间，该存储引擎默认使用HASH索引，也可以在创建表时使用BTREE索引。虽然该存储引擎访问速度非常快，但是一旦数据库发生故障关闭，那么内存中的数据就会丢失。
MEMORY表内容存储在内存中，如果一个表内部变的很大，那么服务器会自动把它转换为一个磁盘表。

创建MEMORY型数据表并制定索引为hash

```
create table tablename(
  id int,
  index using hash(id)
)engine=memory;
```

删除hash索引

```
drop index id on tablename;
```

然后修改索引

```
create index id_index using btree on tablename(id);
```

通过如下命令查看索引

```
show index from tablename \g;
```

也可以在创建表的时候指定表的最大行数

```
create table tablename(
  id int primary key,
)engine=memory max_rows=10000;
```

### 2.4 MERGE
MERGE存储引擎是一组MyISAM表的组合，即将一组结构相同的MyISAM表组合为一个逻辑单元，MERGE表本身没有数据。
对于MERGE表的插入操作，是通过INSERT_METHOD字句完成，可以使用FIRST或者LAST值，其实质上是对MyISAM表进行操作。

```
create table table_myisam_1(
  id int primary key,
  data datetime
)engine=myisam default charset=gbk;


create table table_myisam_2(
  id int primary key,
  data datetime
)engine=myisam default charset=gbk;


create table table_merge_12(
  id int primary key,
  data datetime
)engine=merge union=(table_myisam_1,table_myisam_2) insert_method=first;
```