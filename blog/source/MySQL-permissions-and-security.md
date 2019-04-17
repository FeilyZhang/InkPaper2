title: "MySQL权限与安全"
date: 2019-04-16 22:37:50 +0800
update: 2019-04-16 22:49:50 +0800
author: me
cover: "-/images/mysql-logo.png"
tags:
    - MySQL
preview: MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库中，由mysql_install_db脚本初始化。存储账户权限信息表主要有:user、db、host、tables_priv、columns_priv和procs_priv。

---

## 一、权限表

MySQL服务器通过权限表来控制用户对数据库的访问，权限表存放在mysql数据库中，由mysql_install_db脚本初始化。存储账户权限信息表主要有:user、db、host、tables_priv、columns_priv和procs_priv。

### 1.1 user表

user表的字段可以分为四类，分别是用户列、权限列、安全列和资源控制列。

user表的用户列包括Host、User、Password，分别表示主机名、用户名和密码，其中Host和User为User表的联合主键。当用户与服务器之间建立连接时，输入的账户信息中的用户名称、主机名和密码必须匹配User表中的对应字段，只有三个值都匹配的时候，才允许连接的建立。

权限列的字段决定了用户的权限，描述了在全局范围内允许对数据和数据库进行的操作。包括查询权限、修改权限等普通权限，还包括关闭服务器、超级权限和加载用户等高级权限。普通权限用于操作数据库，高级权限用户数据库管理。user表中对应的权限是针对所有用户数据库的，这些字段值的类型为ENUM，可以取的值只能为Y和N。

安全列只有6个字段，其中两个是ssl相关的，用于加密;2个是x509相关的，用于标识用户;另外两个是授权插件相关的，用于验证用户身份，如果该字段为空，那么服务器使用内建授权验证机制验证用户身份。可以通过`show variables like 'have_openssl'`语句查询服务器是否支持ssl功能。

资源控制列的字段用来限制用户使用的资源，包含四个字段，分别是

+ max_questions:用户每小时允许执行的查询操作次数;
+ max_updates:用户每小时允许执行的更新操作次数;
+ max_connections:用户每小时允许执行的连接操作次数;
+ max_user_connections:用户允许同时建立的连接次数。

### 1.2 db表和host表

db表和host表是MySQL数据库中非常重要的权限表，db表中存储了某个主机对数据库的操作权限，决定了用户能从哪个主机存取哪个数据库，host表中存储了某个主机对数据库的操作权限，配合db权限表对给定的主机上的数据库级操作权限做更细致的控制，这个权限表不受grant和revoke语句的影响。db表比较常用，host表一般很少用。db表与host表结构相似，字段大致分为两类:用户列和权限列。

db表用户列只有三个字段，分别是Host、User、Db，标识从某个主机连接某个用户对某个数据库的操作权限，这三个字段的组合构成了db表的主键。host表不存储用户名称，只有Host和Db两个字段，该表很少用到，一般情况下db表就可以满足权限控制需求了。

db表和host表的权限列大致相同，表中create_routine_priv和alter_routine_priv字段表明用户是否有创建和修改存储过程的权限。

user表中的权限是针对所有数据库的，如果希望用户只对某个数据库有操作权限，那么需要将user表中对应的权限设置为N，然后在db表中设置对应数据库的操作权限。例如，有一个名称为Zhangting的用户分别从名称为large.domain.com和small.domain.com的两个主机连接并操作books数据库，那么可以将用户名称Zhangting添加到db表中，而db表中的host字段值为空，然后将两个主机地址分别作为两条记录的host字段值添加到host表中，并将两个表的数据库字段设置为相同的值books。当用户连接MySQL服务器时，db表中没有用户登录的主机名称，则MySQL会从host表中查找相匹配的值，并根据查询的结果决定用户的操作是否被允许。

### 1.3 tables_priv表和columns_priv表

tables_priv表用来对表设置操作权限，columns_priv用来对表的某一列设置权限。

tables_priv表的字段名分别为:Host、User、Db、Table_name、grantor(表示修改该记录的用户)、Timestamp(表示修改该记录的时间)、Table_priv(表示对表的操作权限，包括Select、Update、Insert、Delete、Create、Drop、Grant、References、Index、Alter等)、Column_priv(表示对表中列的操作权限，包括Select、Insert、Update、Refenrences)。

Columns_priv表字段分别为Host、User、Db、Table_name、Column_name(指定对哪些数据列具有操作权限)、Timestamp、Column_priv。

### 1.4 procs_priv表

该表可以对存储过程和存储函数设置操作权限，包含8个字段，分别是Host、Db、User、Routine_name(表示存储过程或函数的名称)、Routine_type(表示存储过程或函数的类型，有两个值，分别是PROCEDURE和FUNCTION，前者表明是存储过程，后者表明是函数)、Grantor(插入或修改该记录的用户)、Proc_priv(表示拥有的关权限，分别为Execute、Alter Routine、Grant)、Timestamp(表示记录更新时间)。

注意:由于权限信息数据量比较小，访问频繁。所以MySQL服务器在启动后会将权限表的信息缓存起来，所以手动修改权限信息后，需要执行`FLUSH PRIVILEGES`指令来重新刷新缓存中的权限信息。

## 二、账户管理

### 2.1 登录和退出MySQL服务器

```
MySQL [-h localhost] -u root -p [databases_name] [-e "sql;"]
```

以上命令如果加上了-e参数执行SQL语句，那么登录后执行SQL语句后会退出MySQL服务器。

### 2.2 新建普通用户

有两种方式，一种是使用`CREATE USER`或`GRANT`语句，另一种是在根用户下执行SQL语句直接操作MySQL授权表以插入用户。

```
CREATE USER 'jeffrey'@'localhost' [IDENTIFIED BY 'mypass'];
```

省略以上语句的括号部分代表创建的用户登录不需要密码。不省略括号部分那么上述语句指定的是明文密码值，为了避免使用明文密码，可以通过`PASSWORD`关键字使用密码的哈希值设置密码，先通过`PASSWORD`关键字获取密码哈希值，如下

```
select password('mypass');
```

然后在将得到的哈希值替换上述语句的明文密码。

`GRANT`方式创建普通账户命令如下

```
GRANT SELECT,UPDATE ON *.* TO 'testUser'@'localhost' IDENTIFIED BY  'testPswd';
```

两个命令的区别是，`CREATE USER`命令创建用户后用户并没有权限，还需要通过`GRANT`语句赋予用户权限，而`GRANT`语句创建用户可以直接赋予用户权限。

直接操作mysql用户表创建用户，即使用`insert`语句向mysql.user表中增加一条记录，如下

```
INSERT INTO mysql.user(Host, User, Password, [privilegelist]) VALUES('host', 'username', PASSWORD('pswd'), [privilegevaluelist]);
```

### 2.3 删除普通用户

两种方式，分别是通过`DROP USER`语句删除和通过`DELETE`语句删除，如下

```
DROP USER 'jeffrey'@'localhost';
DELETE FROM mysql.user WHERE host = 'localhost' and user = 'customer1';
```

### 2.4 root用户修改自己的密码

一种方式是直接使用`mysqladmin`命令在命令行指定新密码，如下

```
mysqladmin [-h localhost] -u root -p password 'newpswd';
```

一种方式是使用SET语句来修改，需要注意的是新密码必须加密，如下

```
SET PASSWORD=PASSWORD('newpswd');
```

另一种但是是直接操作user表，如下

```
UPDATE mysql.user set Password = password('newpswd') WHERE User = 'root' and Host = 'localhost';
```

### 2.5 root用户修改普通用户密码

一种方式是使用`SET`语句，如下

```
SET PASSWORD FOR 'testUser'@'localhost' = PASSWORD('newpswd');
```

一种方式是使用如2.4所示的`UPDATE`语句来修改。另一种方式是使用`GRANT`语句来修改，如下
```
GRANT USAGE ON *.* TO 'testUser'@'localhost' IDENTIFIED BY 'newpswd';
```

### 2.6 普通用户修改密码

直接使用`SET`语句修改，如下

```
SET PASSWORD = PASSWORD('newpswd');
```

### 2.7 root用户丢失密码解决办法

Windows下先停止MySQL服务器进程，然后键入如下命令启动MySQL服务

```
mysqld --skip-grant-tables
```

然后打开另一个命令行窗口，输入不加密码的登录命令

```
mysql -u root
```

然后使用`mysqladmin`命令或者`UPDATE`语句重置root密码，最后重新加载权限表，如下

```
FLUSH PRIVILEGES;
```

修改密码完成后，就可以将输入`mysqld --skip-grant-tables`命令的命令行窗口关掉，然后使用新密码登录。

## 三、权限管理

权限管理主要是对登录到MySQL的用户进行权限验证，MySQL权限系统的主要功能是证实连接到一台给定主机的用户，并且赋予该用户在数据库上的SELECT、INSERT、UPDATE、DELETE权限。

账户权限信息被存储在MySQL数据库的user、db、host、tables_priv、columns_priv、procs_priv表中，在MySQL启动的时候，服务器将这些数据库表中的权限信息读取内存。

### 3.1 授权与撤销授权

授权可以分为多个层级。

全局权限适用于一个给定服务器中的所有数据库，这些权限存储在mysql.user表中。`GRANT ALL ON *.*`和`REVOKE ALL ON *.*`只授予和撤销全局权限。

数据库权限适用于一个给定数据库中的所有目标，这些权限存储在mysql.db和mysql.host表中。`GRANT ALL ON db_name.`和`REVOKE ALL ON db_name.*`只授予和撤销数据库权限。

表权限适用于一个给定表中的所有列。这些权限存储在mysql.tables_priv表中。`GRANT ALL ON db_name.tb_name`和`REVOKE ALL ON db_name.tb_name`只授予和撤销表权限。

列权限适用于一个给定表中的所有列。这些权限存储在mysql.columns_priv表中。当使用REVOKE时，必须指定与被授权列相同的列。

还有子程序层级权限。

以上`ALL`代表的是所有权限，也可以替换为具体的权限。

### 3.2 查看权限

可以通过如下命令查看指定用户的权限信息

```
SHOW GRANTS FOR 'user'@'host';
```

当然也可以使用`SELECT`语句来查询。

### 3.3 访问控制(过程)

当MySQL允许一个用户执行各种操作时，它将核实该用户向MySQL服务器发送的连接请求，然后确认用户的操作请求是否被允许。也就是MySQL的访问控制分为两个阶段:连接核实阶段和请求核实阶段。

1. 连接核实阶段:当连接MySQL服务器时，服务器基于用户的身份以及用户是否能通过正确的密码身份验证来接受或拒绝连接。这一阶段就是user表在发挥作用。

2. 请求核实阶段:建立了连接之后，MySQL服务器对在该连接上的每个请求都要执行核实操作，检查是否有足够的权限来执行它。这一阶段就是user、db、host、tables_priv或columns_priv表在发挥作用。具体过程为，MySQL先检查user表，若指定权限并未在user表中被授权，那么将依次检查db表、tables_priv、columns_priv表，如果所有权限表都检查完毕，但是还没有找到允许的权限操作，那么就返回错误信息，否则就执行请求。

## 四、MySQL的安全问题

### 4.1 操作系统层面

+ 尽量避免以root方式运行MySQL服务器;
+ 尽量关闭不必要的服务(通过端口扫描确定)。

### 4.2 数据库层面

+ 修改root用户口令和删除匿名账号;
+ 设置安全的密码;
+ 禁止远程连接数据库;
+ 限制单个用户连接次数。