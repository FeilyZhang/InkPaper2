title: "MySQL忘记root密码解决办法"
date: 2019-07-14 14:34:49 +0800
update: 2019-07-14 14:34:49 +0800
author: me
cover: "-/images/mysql-logo.png"
tags:
    - MySQL
preview: Linux下MySQL root用户找回密码。

---

首先编辑以下文件

```
[root@izuf622r98ikh3c0432hejz ~]# vi /etc/my.cnf
```

在打开的文件中的`[mysqld]`中末尾另起一行，并追加

```
skip-grant-tables
```

然后`:wq!`保存！

接下来重启MySQL服务

```
service mysqld restart
```

然后进入MySQL控制台，直接回车

```
mysql -uroot -p
```

执行以下命令

```
mysql> update mysql.user set authentication_string=password('your password') where User="root" and Host="localhost";
```

然后刷新系统授权表

```
mysql> flush privileges;
```

再执行如下命令并退出MySQL控制台

```
mysql> grant all on *.* to 'root'@'localhost' identified by 'your password' with grant option;
mysql> exit;
```

再次修改如下文件，删除之前添加的内容

```
[root@izuf622r98ikh3c0432hejz ~]# vi /etc/my.cnf
```

然后保存再次重启MySQL服务

```
service mysqld restart
```