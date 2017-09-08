title: mysql_3 子查询
date: 

categories: 
- mysql
tags:  
- mysql
---

> 注：本文主要参考[http://szysky.com](http://szysky.com)

### 连接查询

概念: 就是将两个或两个以上的表，连接起来，当做一个数据源，并从中去取得所需要的数据。

就是将每一个表的每一行数据两两之间相互对接起来，每次对接的结果都是连接结果的 一行 数据。

没有条件的连接写法:

- `select * from 表1, 表2;`
- `select * from 表1 join 表2;`
- `select * from 表1 cross join 表2;`

**连接基本形式**

在代码级别，连接的基本形式为:

- `表1 [连接形式] join 表2 [on 连接条件];`

如果是3个表，则进一步扩展为:

- `表1 [连接形式] join 表2 [on 连接条件] [连接形式] join 表3 [on 连接条件]`

**连接的分类**

交叉连接

- 其实就是上面的连接形式，–因为没有条件，只是按连接的基本概念，将所有数据行都连接起来的结果。它又叫做‘笛卡尔积’。
- 对于表1 和表2 的连接，交叉结果是: 行数=两个表字段之和； 列数=两个表的乘积数

**内连接**

- `形式: select * from 表1 [inner] join 表2 on 连接条件。`

例如有 `student`表和 `type`表 学生表中包含科目`pro_id`, 而`Type`表有每个`pro_id`字段并对应说明。这时候如果需要类连接查询如下：

`select * from student inner join type on student.pro_id = type.pro_id;`

优化: `SELECT o.* ,t.sex FROM db1.join1 as o inner join db1.join2 as t on o.type = t.type;`

注意: 这种的表和表之间的内连接传，虽然可以提现为表跟表之间的关系– 一般情况下这就是外键的关系，单并不是有外键关系才可以使用。

**左(外)连接 left(outer) join**

- 形式: `表1(左表) left [outer] join 表2(右表) on 连接条件`
- 含义: 就是将两个表的内连接的结果，再加上左边表不符合内连接所设定的条件的那些数据结果。
- `SELECT o.* ,t.sex FROM db1.join1 as o left join db1.join2 as t on o.type = t.type;`

**右(外)连接 right(outer) join**

同左连接–只不过关键字为right(outer)

**全连接 full(outer) join**

在`mysql`中不支持全连接的语法;

含义: 将两个表内连接的结果，加上左边不符合的结果，和右边不符合的结果。

### 子查询

含义: 一个`select`语句，就是一个查询语句

- `select 字段或表达式 from 数据源 where xx条件判断；`

上述`select`部分，`from`部分，`where`部分，往往都是一些数据 或者数据的集合。

`from`部分, 当然就是 表 ，或 表的连接结果，他们也是数据，只是通常为表数据。

例子 `select pro_id, price from product where price > (select avg(price) from product)`

例子的`where`后面括号中的`select`语句就是子查询。

所以可以认为就是在一个查询语句的内部，某些位置，又出现了查询语句。 由此引出了主查询和子查询的概念。

通常，子查询是为主查询服务的，所以子查询获得一定的结果数据之后，才去执行主查询。

形式: `select 字段或表达式或子查询 [as 别名] from 表名或链接结果或子查询 where 字段或表达式或子查询的条件判断。`

**按子查询结果，可分为如下几个**

- 表 子查询

一个子查询返回的结果理论上是多行多列的时候，此时可以当做一个表来使用，通常是放在`from`后面。

- 行 子查询

一个子查询返回的结果理论是一行多列的时候。此时可以当做一个行来使用，通常放在行比较语法上。
行比较语法类似这样: `where row(字段1,字段2) = (select 行子查询)`;

- 列 子查询

一个子查询返回的结果理论上是多行一列的时候，此时可以当做多个值来使用。
类似(5,8,10,11).

- 标量 子查询

一个子查询返回的结果理论上是一行一列的时候，此时可以当做一个单个值使用，类似这种: `select 5 as c1; 或select ... where id = 17, 或select ... where b>8;`

**子查询，按位置(场合)分**

- 作为主查询的结果数据
- 作为主查询的条件数据
- 作为主查询的来源数据

**常见子查询**

**比较运算符总的子查询**

形式: 操作数 比较运算符 (标量子查询)

说明: 操作数，其实就是比较运算符的2个数据之一而已，通常就是一个字段名。

举例:

找出最高价的商品:

- `select * from product where price = (select max(price) from product);`

**使用in的子查询**

以前的in的用法:

- `xx in (值1, 值2, 值3, ...);`

in的子查询为:

- `xx in (列子查询)`

举例: 找出所有列别名称中带电这个字的所有商品。

可以分两步思考:

`select protype_id from product_type where protype_name like %电%;`

为子查询条件 — 以下为主程序 嵌入where其中。

`select * product where protype_id in('此处为上一步');`

**使用any的子查询**

形式: 操作数 比较运算符 `any` (列子查询);

含义: 当某个操作数(字段) 对于该列子查询的任意一个值，只要有一个满足该比较运算符，则就算是满足了条件。

**使用all的子查询**

形式: 操作数 比较运算符 `all` (列子查询);

含义: 当某个操作数(字段) 对于该列子查询的任意一个值，必须全部满足该比较运算符，则就算是满足了条件。

列子查询 子查询只能是一个字段。

举例: 查询出所有非最高价的商品

1. 查询所有的价格 `select price form product;`
2. 在所有的价格中只有最高价会不满足 `<any`(全部价格)，所以可以这样

- `select * from product where price < any(select price from product);`

同理: 求出最高价的商品价格，特征： 大于所有的价格

- `select * from product where price >= all(select price from produc);`

**使用some的子查询**

一句话: `some`是`any`的同义词。

**使用exists的查询**

形式: `where exists (子查询)`

含义: 该子查询如果有数据，则`exists`的结果是`true`，否则就是`false`。

在实际应用中，该子查询往往都不是独立的子查询，而是会需要跟主查询的数据源(表),建立牟宗关系—通常就是连接关系。建立的方式是”隐式的”，既没有在代码上体现关系，单却在内部有其连接的 实质 。

此隐式方式，通常就体现在查询中的where条件语句中，使用了主查询表中的数据(字段)，

问题: 查询商品表中其类别名称中带”电”这个字的所有商品。

`select * from product where exists(select * from product_type where protype_name like '%电%' and protype_id = product.protype_id);`

注意:

1. 这种子查询语句，没法独立存在或者独立运行的，而是必须跟主查询一起使用。
2. 其他子查询，是可以独立运行的，而且会得到一个运行结果。
3. 盖子查询中的条件，应该设定为跟主查询的某个字段有一定的关联性判断。筒仓该判断就是这两个表的 本来该有的脸连接条件。

### 联合查询union

提示:

在`mysql`的手册中，将连接查询(`join`) 翻译为联合查询。而联合查询(`union`),没有明确翻译。

在通常的书籍或文章，`join`被翻译为连接查询；而`union`才被翻译为联合查询。

**基本概念**

将两个具有相同字段数量的查询语句的结果，以 上下堆叠 的方式，合并为一个查询结果

形式: `select 语句1 union [all | distinct] select 语句2;`

此联合查询语句，默认会”自动消除重复行” ，既默认是`distinct`。 如果想要将所有数据都显示(允许重复行),就是用`all`。