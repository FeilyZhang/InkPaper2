title: "MySQL的高级特性"
date: 2019-07-06 10:00:14 +0800
update: 2019-07-06 10:00:14 +0800
author: me
cover: "-/images/mysql-logo.png"
tags:
    - MySQL
preview: MySQL查询缓存、合并表与分区表、事务控制。

---

## 一、MySQL查询缓存

MySQL查询缓存机制简单说就是会缓存SQL语句和查询的结果，如果运行相同的SQL语句，服务器会直接从缓存中取到结果，而不需要再去解析和执行SQL语句。查询缓存会返回最新的数据而不是过期数据，当数据被修改后，在查询缓存中的任何相关数据均被清除。对于频繁更新的表，查询缓存是不合适的，对于一些不经常改变数据且有大量相同SQL查询的表，查询缓存会提高很大的性能。

查看系统查询缓存是否可用，命令为：

```
show variables like 'have_query_cache';
```

查看查询缓存是否开启，命令为：

```
select @@query_cache_type;
```

开启查询缓存，命令为：

```
set session query_cache_type=ON;
```

关闭查询缓存，命令为：

```
set session query_cache_type=OFF;
```

查询数据库分配给查询缓存的内存大小，命令为：

```
select @@global.query_cache_size;
```

设置查询缓存大小，命令为：

```
set @@global.query_cache_size=1000000;
```

该参数如果需要永久修改，需要修改`/etc/my.cnf`配置文件，添加该参数的选项，如下

```
[mysqld]
port = 3306
query_cache_size = 1000000
...
```

如果查询结果很大，那么就有可能无法缓存，需要设置`query_cache_limit`参数的值，该参数用来设置查询缓存的最大值，默认是1MB，查询与修改的命令如下：

```
select @@global.query_cache_limit;
set @@global.query_cache_limit=2000000;
```

如果需要永久修改，仍然需要修改`/etc/my.cnf`配置文件，添加该参数的选项，如下

```
[mysqld]
port = 3306
query_cache_size = 1000000
query_cache_limit = 2000000
...
```

可以使用如下命令查看查询缓存相关的参数：

```
show variables like '%query_cache%';
```

可以通过以下命令查看查询缓存命中的累计次数：

```
show status like 'Qcache_hits';
```

监控和维护查询缓存的命令如下：

+ `flush query cache`：该命令用于整理查询缓存，以便更好地利用查询缓存的内存，这个命令不会从缓存中移除任何数据；
+ `reset query cache`：该命令用于移除查询缓存中的所有查询结果；
+ `show status like 'Qcache%'`：该命令可以监视查询缓存的使用状况，可以计算出缓存的命中率。

## 二、合并表与分区表

#### 2.1 合并表

合并表是通过之前的merge存储引擎将两个MyISAN表合并起来。

MySQL合并表对性能有一定影响：

+ 合并表看上去是一个表，事实上是逐个打开各个子表，这样的情况下，可能会因为缓存过多的表而导致超过MySQL缓存的最大设置；
+ 创建合并表的CREATE语句不会检查子表是否兼容，如果创建了一个有效地合并表后对某个表进行了修改，那么合并表也会发生错误。

#### 2.2 分区表

表分区就是讲一张大表，根据条件分割成若干小表。查看当前数据库是否支持分区的命令如下：

```
show variables like '%partition%`；
```

+ range分区：使用values less than操作符来进行定义，把连续且不相互重叠的字段分配给分区，命令如下

```
create table table_name(
  field...
) partition by range(field)(
  partition p1 values less than(1000),
  partition p2 values less than(2000),
  partition p3 values less than(3000)[,
  partition p4 values less than maxvalue]
);
```

+ list分区：该分区对应字段的值是一个集合而不是一个范围，如下

```
create table table_name(
  field...
) partition by list(field)(
  partition p1 values in(10, 20),
  partition p2 values in(30),
  partition p3 values in(40)
);
```

+ hash分区；
+ 线性hash分区；
+ key分区；
+ 复合分区。

## 三、事务控制

事务通常包含一系列更新操作，这些更新操作是一个不可分割的逻辑工作单元。要么全部成功，要么全部失败。默认情况下，MySQL事务是自动提交的，如果需要通过明确的COMMIT和ROLLBACK再提交和回滚事务，就需要通过明确的事务控制来开始事务。MySQL的默认开启的事务在一定程度上影响性能，比如1000条数据提交会提交事务1000次，事实上手动开启事务只需要一次提交即可。通过如下方式关闭事务自动提交功能

```
set @@autocommit=0;
```

查看自动提交功能是否关闭：

```
show variables like "autocommit";
```

开启事务的命令为：

```
start transaction;
```

提交事务的命令为：

```
commit;
```

回滚事务的命令为：

```
rollback;
```

如果在表的锁定期间，使用`start transaction`命令开启一个新的事务，会造成一个隐含的`unlock tables`被执行，该操作存在一定的隐患。

另外，事务的ACID特性如下：

+ 原子性（Atomicity）：事务具有原子不可分割的特性，要么一起执行，要么都不执行；
+ 一致性（Consistency）：在事务开始和事务结束时，数据都保持一致状态；
+ 隔离性（Isolation）：在事务开始和结束过程中，事务保持着一定的隔离特性，保证事务在不受外部并发数据操作的影响；
+ 持久性（Durability）：事务完成后，数据将会被持久化到数据库中。