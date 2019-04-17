title: "数据备份与恢复"
date: 2019-04-17 22:34:25 +0800
update: 2019-04-17 22:57:25 +0800
author: me
cover: "-/images/mysql-logo.png"
tags:
    - MySQL
preview: 包括数据备份、数据还原、数据库迁移以及导出数据到外部文件。

---

## 一、数据备份

### 1.1 使用mysqldump命令备份

备份单个数据库中的所有表，以booksdb数据库为例

```
mysqldump -u root - p booksdb > C:/backup/dbs.sql
```

备份单个数据库中的某个表，如下

```
mysqldump -u root - p booksdb table_name > C:/backup/dbs.sql
```

备份多个数据库，如下

```
mysqldump -u root - p --databases booksdb booksdb1 > C:/backup/dbs.sql
```

备份所有数据库，如下

```
mysqldump -u root - p --all-databases > C:/backup/dbs.sql
```

### 1.2 直接复制整个数据库目录

这种方法对InnoDB存储引擎的表不适用。使用这种方法备份的数据最好还原到相同版本的服务器中，不同的版本可能不兼容。

### 1.3 使用mysqlhotcopy工具快速备份

这是备份数据库或者单个表最快的途径，但是该工具只能运行在数据库目录所在的机器上，并且只能备份MyISAM和ARCHIVE类型的表。

该工具在Unix系统中使用方法为，比如将test数据库备份到/usr/backup目录下，命令为

```
mysqlhotcopy -u root -p test /usr/backup
```

需要注意的是，要想执行`mysqlhotcopy`，必须可以访问备份的表文件，具有那些表的SELECT权限、RELOAD权限（以便能够执行FLUSH TABLES）和LOCK TABLES权限。

## 二、数据还原

### 2.1 使用MySQL命令还原

```
mysql -u root -p booksDB < C:/backup/dbs.sql
```

### 2.2 使用source命令还原

前提是已经登录MySQL服务器，然后选择一个数据库，再执行命令

```
source C:/backup/dbs.sql
```

### 2.3 直接复制数据库到数据库目录

如果数据库通过复制数据库文件备份，那么就可以使用这种方式。

### 2.4 mysqlhotcopy快速恢复

mysqlhotcopy备份的文件也可以用来恢复数据库，在MySQL停止运行时，将备份的数据库文件复制到MySQL存放数据的位置（MySQL的data文件夹），重新启动MySQL服务器即可。即

```
cp -R /usr/backup/test /usr/local/mysql/data
```

如果以根用户执行该操作，必须指定数据库文件的所有者，输入语句如下

```
chown -R mysql.mysql /var/lib/mysql/dbname
```

执行完该语句，重启服务器，MySQL将会恢复到备份状态。

## 三、数据库迁移

### 3.1 相同版本的MySQL数据库之间的迁移

使用mysqldump和mysql管道命令组合的方式，如下

```
mysqldump -h www.abc.com -u root -ppassword dbname | mysql -h www.bcd.com -u root -ppassword
```

如果要迁移全部数据库，那么使用参数`--all-databases`

### 3.2 不同版本的MySQL数据库之间的迁移

MySQL服务器升级时，先备份数据文件，再停止服务，然后卸载旧版本，并安装新版MySQL，如果想要保留旧版本中的用户访问控制信息，则需要备份MySQL中的mysql数据库，在新版本MySQL安装完成之后，重新读取备份文件

## 四、表的导出与导入

### 4.1 使用SELECT … INTO OUTFILE导出文本文件

该语句的基本格式为

```
SELECT columnlist FROM table WHERE condition INTO OUTFILE 'filename'  [OPTIONS]
```

其中OPTIONS选项为

+ `FIELDS TERMINATED BY 'value'` : 设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符"\t"。
+ `FIELDS [OPTIONALLY] ENCLOSED BY 'value'` : 设置字段的包围字符，只能为单个字符，如果使用了OPTIONALLY，则只有CHAR和VERCHAR等字符数据字段被包括。
+ `FIELDS ESCAPED BY 'value'` : 设置如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认值为"\"。
+ `LINES STARTING BY 'value'` : 设置每行数据开头的字符，可以为单个或多个字符，默认情况下不适用任何字符。
+ `LINES TERMINATED BY 'value'` : 设置每行数据结尾的字符，可以为单个或多个字符，默认值为"\n"。

### 4.2 使用mysqldump命令导出文本文件

基本格式为

```
mysqldump -T path -u root -p dbname [tables] [OPTIONS]
```

其中OPTIONS选项为

+ `--fields-terminated-by=value` : 设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符；
+ `--fields-enclosed-by=value` : 设置字段的包围字符；
+ `--field-optionally-enclosed-by=value` : 设置字段的包围字符，只能为单个字符，只能包括CHAR和VERCHAR等字符数据字段；
+ `--fields-escaped-by=value` : 控制如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认值为反斜线“\n”；
+ `--lines-terminated-by=value` : 设置每行数据结尾的字符，可以为单个或多个字符，默认值为“\n”。

### 4.3 用MySQL命令导出文本文件

基本格式如下

```
mysql -u root -p --execute="sql" dbname > filename.txt
```

还可以导出为html文件，只需要加上参数`--html`即可，如下

```
mysql -u root -p --html --execute="sql" dbname > filename.html
```

### 4.4 使用LOAD DATA INFILE方式导入文本文件

基本格式为

```
LOAD DATA INFILE 'filename.txt' INTO TABLE tablename [OPTIONS] [IGNORE number LINES]
```

OPTIONS选项包括：

+ `FIELDS TERMINATED BY 'value'` : 设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符"\t"。
+ `FIELDS [OPTIONALLY] ENCLOSED BY 'value'` : 设置字段的包围字符，只能为单个字符，如果使用了OPTIONALLY，则只有CHAR和VERCHAR等字符数据字段被包括。
+ `FIELDS ESCAPED BY 'value'` : 设置如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认值为"\"。
+ `LINES STARTING BY 'value'` : 设置每行数据开头的字符，可以为单个或多个字符，默认情况下不适用任何字符。
+ `LINES TERMINATED BY 'value'` : 设置每行数据结尾的字符，可以为单个或多个字符，默认值为"\n"。

IGNORE number LINES选项表示忽略文件开始处的行数，number表示忽略的行数。

执行LOAD DATA语句需要FILE权限。

### 4.5 使用mysqlimport命令导入文本文件

基本格式如下

```
mysqlimport -u root -p dbname filename.txt [OPTIONS]
```

OPTIONS常用选项包括：

+ `--fields-terminated-by=value` : 设置字段之间的分隔字符，可以为单个或多个字符，默认情况下为制表符；
+ `--fields-enclosed-by=value` : 设置字段的包围字符；
+ `--field-optionally-enclosed-by=value` : 设置字段的包围字符，只能为单个字符，只能包括CHAR和VERCHAR等字符数据字段；
+ `--fields-escaped-by=value` : 控制如何写入或读取特殊字符，只能为单个字符，即设置转义字符，默认值为反斜线“\n”；
+ `--lines-terminated-by=value` : 设置每行数据结尾的字符，可以为单个或多个字符，默认值为“\n”;
+ `--ignore-lines=n` : 忽略数据文件的前n行。