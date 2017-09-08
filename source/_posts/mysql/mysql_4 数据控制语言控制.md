title: mysql_4 数据控制语言控制
date: 

categories: 
- mysql
tags:  
- mysql
---

> 注：本文主要参考[http://szysky.com](http://szysky.com)

### 数据控制语言 DCL (Data Control Language)

数据控制语言,是用于对`mysql`的用户及其权限进行管理的语句.

**用户管理**

mysql中的所有用户,都存在系统数据库(mysql)中的 `user` 表中–不敢那个数据库的用户,都存储在这里.`(服务器: localhost-->数据库: mysql--> 表 user)`

**创建用户**

形式: `create user '用户名'@'允许登录的地址/服务器' identified by '密码'`

如果想创建一个指定的 `ip` 用户想访问我们本地的数据库这样

```
create user 'extra'@'192.168.x.x' identified by '123';    //这样就创建了一个 extra 的用户  对方访问可以这样<br/>
mysql -h我的主机名 -uextra -p;

```

**删除用户**

`drop user '用户名'@'允许登录的地址或服务器'`

**修改用户密码**

`set password = password('密码');`

`setpassword for '用户名'@'允许登录的地址' = password('密码');`

### 权限管理

`mysql` 数据库,将其中所能做的所有事情否分门别类分配到大约30多个权限中去了,其中每个权限,都是一个单词而已.

- `select`: 代表可以查询数据;
- `update`: 代表可以修改数据;
- `delete`: 代表可以删除数据;
- ……

其中有一个特别的权限 : `all` 设置出 `GRANT OPTION` 之外的所有简单权限.

**授予权限**

形式:

- `grant 权限列表 on 某库.某对象 to '用户名'@'允许登录的位置' [identified by '密码'];`

说明:

1. 权限列表,就是多个权限的名字,相互之间用逗号分开,比如: `select,insert,update`, 也可以写 `all`
2. `某库.某对象`,表示给指定的某个数据库中的某个下级单位附权,下级单位有: 表名,视图名,存储过程名,存储函数名.其中可以使用统配语法–>
3. `\*.\*`: 代表所有的数据库的所有下级单位.
4. 某库.*: 代表指定的该库中的所有下级单位

**剥夺权限**

形式:

- `revoke 权限列表 on 某库.某个对象 from '用户名'@'允许登录的位置'`
- 其含义,跟 `grant` 完全一样;

### 数据事务语言 DDL (Data Transaction Language)

就是用来保证多条增删改语句的执行的一致性,要么都执行,要么都没有执行.

**事务的特点**

- 原子性: 一个事务总的所有语句,应该做到要么全做,要么一个都不做.
- 一致性: 让数据保持逻辑上的合理性,例如一个商品出库时,既要让商品库数量减1,又让让对应用户购物车商品加1.
- 隔离性: 如果多个事务同时并发执行,但每个事务就像各自独立执行一样.
- 持久性: 一个事务执行成功,则对数据来说应该死一个明确的硬盘数据更改,而不仅仅是内存中的变化.

**事务模式**

在我们 `cmd` 命令模式中,是否开启了 一条语句就是一个事务 的这个开关.
默认情况下(安装后) ,这个模式是开启的,称为自动提交模式;我们可以手动把他关闭,那就是’非自动提交模式’

`set autocommit = 0/false` //关闭该模式

**事务执行的基本流程**

1. 开启一个事务: `start transaction;` //也可以写成: `begin`

2. 执行多条增删改语句; //也就是相当于希望这多条语句作为一个不可分割的整体去执行.

3. 判断这些语句执行的结果:

   ```
   if(没有出错){
       commit;        //提交事务;此时就是一次性完成;
   } else{
       roolback;    //回滚事务;此时什么都没有做;
   }

   ```

### mysql 编程

**mysql 编程中语句块包含符**

就是相当于 `js` 或 `php` 中的大括号语法:

```
[标识符 :] begin
    //语句...
end;[标识符];

```

标识符就是定义任意的名字而已,比如:

```
if(条件判断)    
    begin
        //.....
    end;
end if;

if(条件判断)
    A:begin
        //...
    end A;
end if;
A 就是标识符,它的租用是标识该语句块,以期可以在该语句块中使用它--其实就是退出.

```

**流程控制语句**

1.`if` 语句

```
if 条件语句 then
    begin
        语句块
    end;

    elseif 条件语句 then 
    begin
        语句块
    end;

    else
    begin
        else 语句块
    end;
end if;

```

2.`case` 语句 相当于 `java` 的 `switch`

```
CASE case_value

    WHEN when_value THEN statement_list

    WHEN when_value1 THEN statement_list    //可以有很多 when 和 then 组合
    ELSE statement_list

END CASE

```

语法 `case @v1 ... end case; //@v1表示一个变量;后面都这样`

**3.loop 循环语句**

```
标识符: loop
begin
    //这里就是循环的语句块
    //注意这里必须要有一个退出循环的逻辑机制,否则该循环就是死循环,其基本形式类似这样
        if(条件) then
            leave 标识符;    //退出
        end if;
end;
end loop 标识符;

```

**4.while循环语句**

```
set @v1<10 do
    begin
        insert into tab1 (id,num) values(null, @v1);
        set @v1 = @v1 + 1;
    end;
end while;

```

**5.repeat语句**
类似于`dowhile`语句,但是这里条件为真就不执行,为假就继续执行.

```
set @v1 = 1;    //赋值语句
repeat
    begin
        insert into tab1 (id,num) values(null,@v1);
        set @v1 = @v1+1;
    end;
    until @v1 >= 9;
end repeat;

```

**leave语句**

语法: `leave` 标识符;

作用: 用来退出`begin...end`结构或其他具有标识符的结构.

**mysql中的变量**

`mysql`中,有两种变量形式:

- 普通变量: 不带 `@` 符号

形式: `declare 变量名 类型名 [default 默认值]; //普通变量必须先这样定义`

使用场景:只能在编程环境使用,有3个:1函数内部,存储过程内部,触发器的内部.

- 会话变量: 带 `@` 符号

形式: `set @变量名 = 值; //跟php类似,无需定义,直接赋值,第一次就算是定义`

使用场景: 任何场景.

**变量赋值**

1. `set 变量名 = 表达式;` #此语法中的变量必须先使用declare声明.
2. `set @变量名 = 表达式;` #此方式可以无需`declare`语法声明,而是直接赋值,类似`php`定义变量并赋值.
3. `select @变量名:=表达式;` #此语句会给该变量赋值,同时还会作为一个`select`语句输出结果集.
4. `select 表达式 into @变量名;`#此语句虽然看起来是`select`语句,但其实并不输出结果集,而只是给变量赋值.

**(存储)函数**

函数,也说成存储函数,其实就是`js`和`php`中所说的函数.

注意:

1. 在函数内部,可以有各种变量和流程控制的的使用.
2. 在函数内部,也可以有各种增删改语句.
3. 在函数内部,不可以用`select`或其他返回结果集的查询类语句.

唯一的区别:

- 这里的函数必须返回一个数据(值);

```
//创建一个函数
create function getMaxValue(p1 float, p2 float, p3 float)
returns float;    #返回float类型
begin
    .... # 要执行的内容
end;

```

上处代码直接在命令行执行可能会有error,因为代码中包含; 会提前结束没有完成的语句.解决方案可以通过修改’语句结束符’. `delimiter ///`

**删除函数**

`drop function`

### 存储过程

存储过程,其实还是函数 – 但其规定: 不能有返回值.

创建一个存储过程

效果目标: 将3个数据写入到表`tab_int`

并返回该表的第一个字段的前3大值得行.

```
create procedure insert_get_date(p1 int, p2 tinyint, p3 bigint)
begin
    insert into tab_int(f1,f2,f3) values(p1,p2,p3);
    select * from tab_int order by table_int.f1 desc limit 0,3;
end;

```

调用存储过程:

- `call` 存储过程名(实参1, 实参2, …)
- 它应该是在非编程环境中调用,既执行增删改查的场景.

**删除存储过程**

`drop procedure 存储过程名;`

### 触发器 trigger

触发器,也是一段预先定义好的编程代码(跟存储过程和存储函数一样),并有个名字.

它不能调用,而是,在某个表发生某个事件(增,删,改)的时候,会自动触发而调用起来.

定义形式:

- `create trigger 触发器名 触发时机 触发事件 on 表名 for each row`

begin //…. end;

说明:

1. 触发时机,只有两个: `before`(在…之前), `after`(在…之后);
2. 触发事件,只有三个: `insert`, `update`, `delete`
3. 既触发器的含义是: 在某个表上进行`insert`(或`update`或`delete`)之前,或之后.
4. 通常,触发器用于在对某个表进行增删改的操作的时候,需要同时去做另外一件事的时候;
5. 在触发器的内部,有两个关键字代表某种特定的含义,可以用来获取数据
   `new`:它代表当前正要执行的`insert`或`update`的时候’新行’数据;通过他,可以获取这一新行数据的任意一个字段形式为: `set @v1 = new.id;` //获得该新插入或update行的id的值(前提是有该id).
6. `old`:它代表当前正要执行的delete的时候’旧行’数据;通过他,可以获取这一旧行数据的任意一个字段形式为: `set @v1 = new.id;` //获得该新插入或`update`行的`id`的值(前提是有该id).