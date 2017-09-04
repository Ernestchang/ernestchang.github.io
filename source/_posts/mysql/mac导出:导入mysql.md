title: mac导出/导入mysql
date: 
categories: 
- mysql 

tags:  
- mysql
---

## mac 安装mysql

 首先 `$ brew install mysql` 安装 mysql，之后可能需要重启一下终端，然后成功的话输入 `$ mysql` 再按 Tab 会提示一些和 mysql 相关的命令出来，如果有的话就 OK 了，如下：

```
$ mysql
mysql                       mysql_zap
mysql.server                mysqladmin
mysql_client_test           mysqlbinlog
mysql_client_test_embedded  mysqlbug

```

## 配置和使用

MySQL 和直接使用的 SQLite 不同，需要有一个 MySQL server 在，之后通过 socket 去连接上 MySQL，然后再进行对数据库的相关操作。

### 启动服务和配置 root 用户
1. 靠下面的命令来启动服务器。

```
$ mysql.server start
Starting MySQL
.. SUCCESS!

```

2. 关闭服务器的命令也很简单：

```
$ mysql.server stop
Shutting down MySQL
.. SUCCESS!

```

之后重新开启来进行后面的操作。

3. 设置初始密码。MySQL 也有自己的用户系统，所以后面提到的用户不要和操作系统的弄混了。默认的用户是 root，没有密码。可以通过下面命令来设置一个密码：

```
$ mysqladmin -u root password
New password:
Confirm new password:

```

4. 修改密码。包括 `mysqladmin` 在内，其他的 mysql 需要指明用户的时候加 `-u username`，如果该用户设置过密码需要添加 `-p` 指令，所以如果已经设置过 root 用户的密码了之后再想修改密码，用前面的指令会报错，如下：

```
$ mysqladmin -u root password
mysqladmin: connect to server at 'localhost' failed
error: 'Access denied for user 'root'@'localhost' (using password: NO)'

```

这时候只需要加上 `-p` 选项就好。

```
$ mysqladmin -u root password -p
Enter password:
New password:
Confirm new password:

```

### 使用 root 用户连接数据库

上面 root 用户已经 OK 了，赶快连接下数据库吧。

```
$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 5.6.26 Homebrew

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```

接着执行一下命令，进行一些简单操作。

```
mysql> system pwd
/usr/local/Cellar/mysql/5.6.26
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
mysql> CREATE DATABASE mydb;
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mydb               |
| mysql              |
| performance_schema |
| test               |
+--------------------+
mysql> use mydb;
Database changed
mysql> SHOW TABLES;
Empty set (0.00 sec)

```

### 创建一个新用户

root 用户的权限一般不会随便拿去使用，所以创建一些用户，并分配对应的权限。下面的命令创建的是一个名字是 `user12`密码是 `34klq*` 的本地用户：

```
mysql> CREATE USER user12@localhost IDENTIFIED BY '34klq*';

```

之后给他分配 mydb 数据库的权限：

```
mysql> GRANT ALL ON mydb.* to user12@localhost;

```

## 简单上手使用

- mysql.server start
- 设置 root 密码
- 用 root 登陆，创建表
- 创建新用户分配权限
- 开始简单使用

## 导出/导入

#### 准备：

打开Terminal，输入mysqldump，

发现Terminal提示《mysqldump: command not found》，需配置mysql全局环境变量如以下：

打开terminal输入`vi ~/.bash_profile` 或者`open .bash_profile`添加如下三行代码:

` PATH=$PATH:/usr/local/mysql/bin ` 

保存并退出后在terminal输入source ~/.bash_profile然后就成功了

#### 导出：

     > 打开『终端』输入：
     > cd 『打开要将.sql文件生成的文件位置』
     > 输入：mysqldump -u root -p database_name>sql_name.sql
     > 查看导出过程：tail -f /Users/admin/dump.sql

下面是通过MySQL命令行导出数据库或表的方法：

**MySQL命令行导出数据库：**

`mysqldump -u 用户名 -p 数据库名 > 导出的地址/导出的文件名`

如我输入的命令行:

`mysqldump -u root -p Library > '/Users/lichen/Desktop/Library.sql'`

输入后会让你输入进入MySQL的密码,输入密码即可看到hehe.sql出现在桌面上。

**MySQL命令行导出一个表：**

`mysqldump -u 用户名 -p 数据库名 表名> 导出的地址/导出的文件名`

如我输入的命令行：

`mysqldump -u root -p Library reader> '/Users/lichen/Desktop/Library_reader.sql'`

输入后会让你输入进入MySQL的密码,输入密码即可看到Library_reader.sql出现在桌面上。

#### 导入：

```
> 打开终端输入：（前提是已经配置过MySQL环境变量）
> mysql -u root -p
> create database name;
> use name;
> source 『将.sql文件直接拖拽至终端，自动补全其文件目录』
```

### 异常处理

1. 报错提示告诉我们，需要重新设置密码。

```
**ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement**
```

那我们就重新设置一下密码，命令如下：

```
set password = password('123456');
```

这次，我们在再次执行 `show databases` 就没有问题了。

### 其他设置

1. 退出mysql提示符

   ```
       mysql> quit
       
       or control-D
   ```

   ​