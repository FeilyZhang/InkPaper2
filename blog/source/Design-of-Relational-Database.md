title: "数据库范式与关系数据库设计以及简单查询和复杂查询"
date: 2019-07-06 15:50:14 +0800
update: 2019-07-06 15:50:14 +0800
author: me
cover: "-/images/mysql-logo.png"
tags:
    - MySQL
preview: 函数依赖与数据库三大范式、关系数据库的设计、简单查询、复杂查询。

---

## 一、数据库范式

+ 函数依赖：一张表中，两个字段值的一一对应关系称为函数依赖；
+ 第一范式：如果一张表内同类字段不重复出现，则该表就满足第一范式的要求；
+ 第二范式：在满足第一范式的基础上，如果每个“非关键字”字段仅仅函数依赖于主键，那么该表就满足第二范式的要求；
+ 第三范式：在满足第二范式的基础上，并且不存在“非关键字”字段函数依赖于任何其他“非关键字”字段，那么该表就满足第三范式的要求。

## 二、关系数据库的设计

+ 为每个实体建立一张数据表；
+ 为每张表定义一个主键；
+ 增加外键表示一对多关系；
+ 而特殊的一对一关系可以通过表示一对多关系的外键增加唯一性约束来实现；
+ 建立新表表示多对多关系，即将两张表的主键分别作为没有添加唯一性约束的外键关联在另一张新表中；
+ 为字段选择合适的数据类型；
+ 定义约束条件：主键约束、外键约束、唯一约束、非空约束、检查约束、默认值约束；
+ 使用规范化减少数据冗余，即满足三大范式。

## 三、简单查询

基本的查询语法格式如下

```
select [* | distinct | distinctrow | col_name...] [from table_name] [where condition] [group by col_name] [having condition] [order by col_name [asc | desc]] [limit [offset,] row_count];
```

+ `[* | distinct | distinctrow | col_name...]`：*代表表中的全部字段；distinct和distinctrow代表的是同一意思，就是去除查询结果中相同的行；col_name代表查询的字段名，多个字段用逗号隔开；
+ `[where condition]`：代表查询的条件，可以使用运算符进行连接；
+ `[group by col_name]`：代表分组的条件，对查询的结果进行分组；
+ `[having condition]`：在分组查询中使用的条件语句，并且只能在分组查询中使用；

#### 3.1 where条件运算符

+ `and`：并且关系；
+ `or`：或者关系；
+ `like`：模糊查询，`_`代表一个字符，`%`代表0到多个字符；
+ `in`操作符：表示在某一范围之内，使用方式为`where 字段 in(..., ...)`, in中也可以是一个查询语句，如`where 字段 in(select * from ...)`;
+ `not in()`操作符：与in相反；

## 四、复杂查询

#### 4.1 分组查询

分组查询通过`select`语句的`group by`子句来完成，通过分组查询可以很容易完成查询中的统计操作，与普通查询不同，分组查询的条件是通过`having`来指定的而不是`where`。

需要注意的是，当在一个查询中使用group by子句时，它的select子句后面只能是聚合函数或者是group by之后的列名，否则查询后的结果没有任何意义，示例如下(按subject字段查询studentinfo表中subject字段的记录数)

```
select subject, count(*) from studentinfo group by subject;
```

也可以对多列进行分组查询。

#### 4.2 多表查询

##### 4.2.1 等值连接

等值连接就是讲多个表之间的相同字段作为条件查询数据，然后将对应的字段值查询出来。通常情况下，多个表之间的相同字段指的是表与表之间的主外键，这样就可以将一对多关系直接全部查询出来，示例如下

```
select newstudentinfo.name, subjectinfo.subjectname from newstudentinfo, subjectinfo where newstudentinfo.subjectid=subjectinfo.id;
```

当然可以通过where连接多个条件来查询多个表的满足条件的记录，如下示例：

```
select newstudentinfo.name, subjectinfo.subjectname, teacherinfo.teachername from newstudentinfo, subjectinfo, teacherinfo where newstudentinfo.subjectid=subjectinfo.id and newstudentinfo.teacherid=teacherinfo.id;
```

##### 4.2.2 笛卡尔积

查询内容是所有查询的数据表中所有列的和以及行的乘积，对实际应用没有太大意义。

##### 4.2.3 外连接

等值连接的查询结果全部是符合条件的行组成的，如果想得到查询结果之外的行应该怎么办？答案是使用外连接查询来完成，外连接分为左外连接和右外连接。

左外连接的查询结果是，除了返回表中符合条件的记录外还要加上左表中剩下的全部记录，右外连接的查询结果是，除了返回表中符合条件的记录外还要加上右表中剩下的全部记录。具体的语法如下所示：

```
select col... from tableA left outer join(right outer join) tableB on condition;
```

##### 4.2.4 内连接

内连接与外连接不同，内连接不分左右并且使用内连接查询的结果都是符合条件的结果。与上面的等值连接是很相似的，语法如下

```
select col... from tableA inner join tableB on condition;
```

内连接与等值连接相比，内连接的好处就是可以更好地明确数据表的连接方式，同时，使用`on`作为连接条件也能更好地清楚地知道使用的是多表连接。

#### 4.3 合并查询结果

合并查询结果就是讲两张表的查询结果垂直合并在一起，前提是两张表查询字段的数据类型必须一致，语法格式如下

```
select col... from table_name union select col... from table_name;
```

还可以对查询的结果进行排序，如下

```
(select col... from table_name) union (select col... from table_name) order by col_name;
```

还可以限制组合查询结果的行数

```
(select col... from table_name) union (select col... from table_name) limit 行数;
```