title: "CentOS 7.3安装Redis并开启开机启动，并远程连接"
date: 2019-07-14 16:36:49 +0800
update: 2019-07-14 16:36:49 +0800
author: me
cover: "-/images/redis.jpg"
tags:
    - Redis
preview: Linux CentOS 7.3下安装Redis并开启开机启动，并远程连接

---

### 一、安装Redis

下载并解压Redis安装包到指定目录

```
wget http://download.redis.io/releases/redis-4.0.9.tar.gz  
tar -zxvf redis-4.0.9.tar.gz -C /usr/local/
```

接下来安装gcc以来，先检查有无以来，没有则继续执行下一条命令

```
gcc -v
yum install -y gcc
```

然后cd到redis的解压目录下，进行编译安装

```
cd /usr/local/redis-4.0.9/
make MALLOC=libc
cd src && make install
```

然后测试是否安装成功

```
cd /usr/local/redis-4.0.9/src/
./redis-server
```
如果出现提示信息则说明安装成功。接下来配置开机启动

## 二、配置开机自启

修改`/usr/local/redis-4.0.9/redis.conf`文件中的`daemonize no`为`yes`

然后指定redis.conf文件启动：`./redis-server /usr/local/redis-4.0.9/redis.conf`

由于上面我们执行了redis进程启动，通过`ps -aux | grep redis`查看redis进程，并用`kill -9 进程id`杀死,然后执行下述操作：

+ 在/etc目录下新建redis目录，`mkdir /etc/redis`;
+ 将/usr/local/redis-4.0.9/redis.conf 文件复制一份到/etc/redis目录下，并命名为6379.conf, `cp /usr/local/redis-4.0.9/redis.conf /etc/redis/6379.conf`;
+ 将redis的启动脚本复制一份放到/etc/init.d目录下,`cp /usr/local/redis-4.0.9/utils/redis_init_script /etc/init.d/redisd`;
+ 设置redis开机自启动:
  + 先切换到/etc/init.d目录下,然后执行自启命令`chkconfig redisd on`；
  + 如果显示`service redisd does not support chkconfig`，解决方法为，使用vim编辑redisd文件，在第一行加入如下两行注释，保存退出
```
# chkconfig:   2345 90 10
# description:  Redis is a persistent key-value database
```
  + 再次执行开机自启命令`chkconfig redisd on`，这个时候应该就能成功了。

现在可以直接已服务的形式启动和关闭redis了；

+ 启动：`service redisd start`；
+ 关闭：`service redisd stop`.

## 三、远程连接

因为redis默认设置允许本地连接，所以我们要将`redis.conf`中将`bind 127.0.0.1`改为`bind 0.0.0.0`或者注释该行;

另外，阿里云ECS有一个安全组，找到并添加规则允许6379端口访问。

设置redis连接密码方式如下：

在`redis.conf`中搜索`requirepass`(502行左右)这一行，然后在合适的位置添加配置`requirepass yourpassword`;

设置完成后执行`/usr/local/bin/redis-server /usr/local/redis-4.0.9/redis.conf`更新配置。

然后测试看能不能ping通目标IP与端口

```
C:\Users\Administrator
λ tcping ip 6379

Probing ip:6379/tcp - Port is open - time=138.879ms
Probing ip:6379/tcp - Port is open - time=61.759ms
Probing ip:6379/tcp - Port is open - time=1087.038ms
Probing ip:6379/tcp - Port is open - time=65.800ms

Ping statistics for ip:6379
     4 probes sent.
     4 successful, 0 failed.  (0.00% fail)
Approximate trip times in milli-seconds:
     Minimum = 61.759ms, Maximum = 1087.038ms, Average = 338.369ms
```

然后试一下远程连接

```
[root@izuf622r98ikh3c0432hejz src]# ./redis-cli -h ip -p 6379 -a pswd.
ip:6379>
```

成功！