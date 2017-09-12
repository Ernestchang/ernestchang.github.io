title: 《Android 编程实战》Chap7_序列化说明
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---
> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## 数据存储的介绍

谈到数据数据存储通常会使用`持久化`, 而用`序列化`描述数据是如何表现其存储状态的. 如果没有数据的`持久化`, 那么数据还能在`RAM`中保持其状态, 一旦相关进程结束数据就会消失. 实现数据的持久化通常涉及性能, 延迟, 数据大小和复杂度等因素的这种. 例如, 快速的数据读取往往会导致较慢的写入. 序列化就是关于数据如何组织的, 同时包括在持久化状态和内存中.

## Android持久化选项

`Android`中提供了两种现成的方法, 分别是:

- `偏好文件(preference files)`: 使用`XML`格式, 通过`SharePreferences`类提供接口
- `数据库(sqlite)`: 通常被包装成`ContentProider`组件.

通常选项, 应用配置属性用`preferences files`来存储; 数组表格,或者Java表示的一些数据用数据库进行存储; 而一些二进制数据, 例如图片通常当做常规文件来存储在本地.

## Preferences file

这些文件存储在应用程序的内部目录中, 其结构只允许存储键值对.

创建`SharePreferences`对象最简单的方式是使用`PreferenceManager.getDefaultSharedPreferences()`方法, 它会返回应用程序默认的偏好对象. 这种方式的便利是系统会自动管理**偏好文件名**. 如果需要多个偏好文件那么要使用`Context.getSharedPreferences()`方法, 它允许开发者自由命名文件. 如果只是创建`Activity`相关的偏好文件, 可以使用`Activity.getPreference()`方法, 他会在调用时得到`Activity`的名字.

> `getDefaultSharePreferences()`创建的偏好文件名是由包名+`_preferences`组成的.

其内部支持的存储值类型有`int`, `float`, `long`, `boolean`, `String`以及`Set<String>`对象. 键名必须是一个有效的字符串.

------

**如果需要文件修改并提交存储**

首先需要获得`Editor`实例, 他提供了相应的`PUT`方法, 以及用于提交修改的方法. 在`Android 2.3`之前, 通过使用`commit()`方法把修改**同步**提交到存储设备中. 但在2.3之后, `Editor`类提供了用于**异步**执行写操作的`apply()`方法. 因为要尽可能地避免主线程执行阻塞操作, 所以`apply()`相对来说比`commit()`更好.

------

**如果需要对偏好值被修改的时候可以收到通知**

那么可以通过注册一个回调函数, 每当`apply()`和`commit()`方法时都会触发该监听器. 通过对`SharedPreferences`的实例调用`registerOrSharedPreferenceChangeListener()`方法来添加内容改变回调.

### 用户选项和设置用户界面

Android提供一套现成的`Activity`和`Fragment`类针对用户更改应用程序的选项和设置, 使得创建这类用户界面非常简单容易`PreferenceActivity`和`PreferenceFragment`.

但! 是! 感觉没啥用处, 基本应用都会按照自己的ui风格进行设计. 知道一下就行

## ContentProvider注意事项

### Android数据库设计

关系数据库的设计通常通过`数据库规范化`完成. 该过程使用一些`范式`规则来减少数据库中的依赖和冗余. 有许多数据库范式, 但在大多数情况下, 只有前三个是相关的. 如果一个数据库设计满足了前三个范式, 可以认为它是`规范化`的.

由于面向的对象为`android`应用. 有可能不一定在表中尽可能多的使用外键, 虽然使用外键可以更好的减少空间的占用, 但是相对于开发中操作多表的困难度也就相应的提高. 所以需要权衡一下.

### 创建和升级数据库

建议总是使用`ContentProvider`组件包装`SQLite`数据库. 通过这种方式, 可以只在一个地方管理的数据库调用, 还可以使用一些现成的数据库工具类.

### 查询方法说明

查询数据库(通常会调用`ContentResolver.query()`)会调用`ContentProvider.query()`方法. 在实现查询方式时必须解析传入的`Uri`以决定执行哪个查询, 并且还要检查所有传入的参数是否正确.

> 编写数据库查询要把`WHERE`语句中较简单的比较放在前面. 这样会加快查询, 因为其可以尽早决定要包含的信息.

### 数据库事务

每次在`SQLite`数据库执行一条`SQL`语句都会执行一次数据库事务操作. 除非是自己专门管理事务, 否则每条语句都会自动创建一个事务. 因为大多数`ContentProvider`调用最终都只会生成一条`SQL`语句, 这种情况下几乎没有必要手动处理事务. 但是如果应用程序将执行多条`SQL`语句, 比如一次插入很多条记录, 记得总是自己管理事务.

`ContentProvider`类提供了两个事务管理的方法: `ContentProvider.bulkInsert()`和`ContentProvider.applyBatch()` 相比与普通的`insert()`快了很多.

事务的语义很简单. 首先调用`SQLiteDatabase.beginTransaction()`开始一个新的事务. 当成功插入所有记录之后调用`SQLiteDatabase.setTransactionSuccessful()`, 然后使用`SQLiteDatabase.endTransaction()`结束本次事务. 如果某条数据插入失败, 会抛出`SQLiteException`, 而之前的所有插入都会回滚, 因为在成功之前没有调用过`setTransactionSuccessful()`

**bulkInsert()虽然会提高数据插入性能, 但是此方法只使用插入操作**

如果要在一次事务中执行多次`update()`或者`delete()`语句, 必须实现`ContentProvider.applyBatch()`方法. 接收一个`ContentProviderOperation集合`.内部循环集合通过调用`apply()`在事务内部实现操作.

该API是为`ContactsProvider`等较复杂的`ContentProvider`设计的, 他们有许多连接的表, 每个都有自己的`Uri`. 另外如果要批量插入多个表, 该API也可以使用.

## 序列化数据

如果要在`Intent`上传输数据或者和另外一台设备共享数据, 需要把数据转化成接收端能识别的格式, 并且还要适于在网络上传输. 这种技术称为`序列化(serialization)`

序列化是从内存中取出数据并把其写到文件(或者其他输出)中, 是的以后能读取完全相同的数据称为`反序列化`. android内部使用了`Parcelable`接口来处理序列化工作, 但它不适合在文件上持久存储或者在网络上传输数据.

### JSON

`JSON`是`JavaScript Object Notation`的缩写, 是`JavaScript`标准的一个子集. 这种格式很适合表示非二进制数据的.

例如一下JSON数据:

```
[
    {
        "name":"张三"
        "id"  :"1"
        "sex" :"男"
    },
    {
        "name":"张三"
        "id"  :"1"
        "sex" :"男"
    },
    {
        "name":"张三"
        "id"  :"1"
        "sex" :"男"
    }
]
```

这里说一下另一个`API`作为了解, 通过获得输入输出流利用`JsonReader`和`JsonWriter`API进行. 相比较直接把一个流中的全部内容读取为一个`String`中, 然后传给`JSONArray`的构造函数, 使用`JsonReader`会消耗更少的内存, 并且相对来说会更快.

直接使用JSON API 现在也不常用, 想了解看一下[实例代码](https://github.com/suzeyu1992/AndroidProgrammingPushingTheLimits/blob/master/StoreOrTest/app/src/main/java/com/szysky/note/storeortest/serialization/JSONDemo.java)

### Gson介绍

`JSONObject`和`JSONArray`类使用起来虽然很方便, 但是他们也有一定的局限性, 并且通常会消耗更多不必要的内存. 同样, 如果有多个不同类型的对象, 使用`JsonReader`和`JsonWriter`需要编写比较多的代码. 所以如果需要更高级的JSON数据序列化和反序列化方法, 可是使用`Gson`

`Gson`允许把简单的Java对象转换成`JSON`, 反之亦然. 所以常做的就是生成一个`JavaBean`对象, 提供set/get并转换成json对象.