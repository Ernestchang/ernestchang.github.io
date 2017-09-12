title: 面试之Java篇
date: 

categories: 
- java
- 面试
tags:  
- java
---

> 注：本文主要参考[http://szysky.com](http://szysky.com)

## Java基础

### 关键字transient volatile

**transient**

> 加了该关键字的属性，不会被序列化。

换句话说，这个字段仅存于调用者内存，而不会写到磁盘里持久化。
序列化原理：将待续列化的对象中的信息写入到磁盘或网络流中。序列化中的引用会进行“深度复制”，并且如果两个对象有共同的引用对象，且两个对象都写入同一个流，那么该引用对象不会重复创建，只会创建一次，并还原到虚拟机后引用的还是同一个对象。但是，如果写入不同的流中，那么两次创建的是完全不同还原对象。

**volatile**

每个线程访问堆中对象时，将堆中对象`load`到线程本地内存中，建立一个变量副本，之后线程就不再和对象在堆变量值有任何关系，而是直接修改副本变量值。
修改完之后，自动把线程变量副本的值写到对象在堆变量中。这样堆中对象的值就产生了变化。这些操作不是原子性的。
使用volatile修饰变量，JVM只是保证从内存加载到线程工作内存中的值是最新的。因此，即使使用volatile还是会存在并发情况。
比如：

```
volatile static int a=0;
//线程A和线程B同时执行
a++;
```

此时线程A拿到a的最新值0，线程B也拿到最新值0；但是，A执行a++后，值为1，B也同样计算得到a=1，它们再同时写回到堆内存，使得最后a的值为1，并不为2.

### Java1.7 1.8新特性

**Java1.7**

（1）所有整数int， short，long，byte都可以用二进制表示，用0b开头。
（2）switch支持String类型。
（3）try-with-resource语句：在try()里面（括号里面）语句中声明一个或多个资源，try括号中的资源在最后会自动关闭.
（4）catch多个异常
（5）数字类型的下划线表示 更友好的表示方式，不过要注意下划线添加的一些标准，如:
`long creditCardNumber = 1234_5678_9012_3456L;`
(6)泛型实例的创建可以通过类型推断来简化 可以去掉后面new部分的泛型类型，只用<>就可以了

------

**Java1.8**

(1)lambda表达式，最大的新增的特性
(2)允许给接口添加非抽象（即子类可以不用去实现）的方法，需要在方法前加default
(3)函数式接口。每个lambda表达式都对应一个类型，通常是接口类型。而“函数式接口”是指仅仅只包含一个抽象方法的接口，每一个该类型的lambda表达式都会被匹配到这个抽象方法。因为默认方法不算抽象方法，所有也可以给函数式接口添加默认方法。我们可以将lambda表达式当成任意一个只包含一个抽象方法的接口类型。为确保你的接口满足这个要求，可以添加@FuntionalInterface注解。
(4)方法与构造函数引用。Java8允许使用`::`关键字来传递方法或者构造函数的引用。
(5) ……

### interface和abstract类区别

1. 继承方面: `abstract class`在Java中表示的是一种继承关系，一个类只能使用一次继承关系。但是，一个类却可以实现多个`interface`。
2. 成员变量方面: 在`abstract class`中可以有自己的数据成员，也可以有`非abstarct`的方法，而在`interface`中，**只能够有静态的不能被修改的数据成员**（也就是必须是`static final`的，不过在`interface`中一般不定义数据成员），所有的方法都是`public abstract`的。
3. 抽象方法方面: 实现抽象类和接口的类必须实现其中的所有抽象方法。抽象类中可以有非抽象方法，而接口中所有方法为抽象方法。
4. 访问权限方面: 抽象类中的变量默认是`friendly`型，其值可以在子类中重新定义，也可以重新赋值。接口中定义的变量默认是`public static final`型，且必须给其赋初值，所以实现类中不能重新定义，也不能改变其值。
5. 设计理念方面: `abstract class`和`interface`所反映出的设计理念不同。其实`abstract class`表示的是”`is-a`“关系，`interface`表示的是”`like-a`“关系。

### XML解析方式DOM,SAX,PULL

**DOM**

通过DOM解析xml的好处就是:
我们可以随时访问到某个节点的相邻节点，并且对xml文档的插入也非常的方便

不好的地方就是:
其会将整个xml文档加载到内存中，这样会大大的占用我们的内存资源
对于手机来说，内存资源是非常非常宝贵的，所以在手机当中，通过DOM这种方式来解析xml是用的比较少的。使用DOM方式，类似JS，可以调用getElementsByTagName()、getChildNodes()等等方法。

------

**SAX**

`SAX`解析`xml`是基于事件流的处理方式的。因此每解析到一个标签，它并不会记录这个标签之前的信息，而我们只会知道当前这个标签的名字和它的属性，至于标签里面的嵌套，上层标签的名字这些都是无法知道的。SAX解析xml最重要的步骤就是定义一个我们自己的`Handler`处理类，我们可以让其继承 `DefaultHandler`这个类，然后在里面重写5个回调方法，分别是：

- startDocument
- startElement
- characters
- endElement
- endDocument

------

**PULL**

`Pull`解析和`SAX`解析类似，都是基于事件流的方式，在`Android`中自带了`Pull`解析的jar包，所以我们不需要导入第三方的jar包了。Pull解析器和SAX解析器虽有区别但也有相似性。

**他们的区别为：**
`SAX`解析器的工作方式是自动将事件推入注册的事件处理器进行处理，因此你不能控制事件的处理主动结束；而`Pull`解析器的工作方式为允许你的应用程序代码主动从解析器中获取事件，正因为是主动获取事件，因此可以在满足了需要的条件后不再获取事件，结束解析。

**他们的相似性在运行方式上:**

> Pull解析器也提供了类似SAX的事件（开始文档`START_DOCUMENT`和结束文档`END_DOCUMENT`，开始 元素 `TART_TAG`和结束元素`END_TAG`，遇到元素内容`TEXT`等），但需要调用`next()`方法提取它们（主动提取事件）。调用parser.nextText();方法获取标签内的文本

```
XmlPullParserFactory factory = XmlPullParserFactory.newInstance(); 
XmlPullParser xmlPullParser = factory.newPullParser(); 
xml.setInput(new StringReader(xmlData)); 
 
int eventType = xmlPullParser.getEventType(); 
 
while(eventType!=XmlPullParser.END_DOCUMENT){ 
    String nodeName = xmlPullParser.getName(); 
    switch(eventType){ 
        case XmlPullParser.START_DOCUMENT:{} 
        case XmlPullParser.START_TAG:{} 
        case XmlPullParser.END_TAG:{}  
    }  
    eventType = parser.next(); 
}
```

### foreach和for循环

**foreach**

foreach本质是通过迭代器遍历，有如下特点：

- 无需获取容器大小
- 需要创建额外的迭代器变量
- 遍历期间得到的是对象，没有索引位置信息，因此没办法将指定索引位置对象替换为新对象

------

**for**

- for需要获取容器大小，如果计算大小比较耗时，那么for循环效率肯定低下
- for循环是根据容器大小防止越界，因此每次循环需要进行一次比较

**效率**

由于每次循环时，使用`for`循环都得计算容器大小并且还需要比较，因此，在对容器里面的每个元素进行遍历时，`foreach`效率更高。

这个结论也不是绝对的，在选择`for`，`foreach`的时候，应该考虑以下几点：

- 如果只是读数据，优先选择foreach，因为效率高，而且代码简单，方便；
- 如果要写数据，即替换指定索引位置处的对象，就只能选择for了，而且选择第二个for效率更高！

### NIO

> NIO是非阻塞的IO，Java NIO由一下几个核心部分组成：`Channels`、`Buffers`、`Selectors`。

虽然Java NIO中除此之外还有很多类和组件，但是Channel，Buffer和Selector构成了核心的API。其他组件如Pipe和FileLock,只不过是与其他三个核心组件共同使用的工具类。

- `Channel`：基本上所有的IO在NIO中都从一个Channel开始，Channel有点像流。数据可以从Channel读到Buffer中，也可以从Buffer写到Channel中。Channel和Buffer有好多类型，Channel主要有：FileChannel、DataGramChannel、SocketChannel、ServerSocketChannel。涵盖了UDP和TCP网络的IO以及文件IO。
- `Buffer：`NIO主要的Buffer有：ByteBuffer、CharBuffer、DoubleBuffer、FloatBuffer、IntBuffer、LongBuffer、ShortBuffer这些Buffer涵盖了你能通过IO发送的基本数据类型。
- `Selector`：允许单线程处理多个Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如一个聊天服务器中。要使用Selector，得先向Selector注册Channel然后调用它的select()方法。这个方法会一直堵塞知道某个注册的通道有事件就绪。一旦这个方法返回线程就可以处理这些事件，事件的例子有如新连接进来，数据接收等。

------

**NIO的使用**

```
// 注册
// 创建Selector对象   
Selector sel = Selector.open();   
// 创建可选择通道，并配置为非阻塞模式   
ServerSocketChannel server = ServerSocketChannel.open();   
server.configureBlocking(false);   
 
// 绑定通道到指定端口   
ServerSocket socket = server.socket();   
InetSocketAddress address = new InetSocketAddress(port);   
socket.bind(address);   
 
// 向Selector中注册感兴趣的事件   
server.register(sel, SelectionKey.OP_ACCEPT);    
return sel;


// 处理
try {    
    while(true) { 
        // 该调用会阻塞，直到至少有一个事件发生 
        selector.select(); 
        Set<SelectionKey> keys = selector.selectedKeys(); 
        Iterator<SelectionKey> iter = keys.iterator(); 
        while (iter.hasNext()) { 
            SelectionKey key = (SelectionKey) iter.next(); 
            iter.remove(); 
            process(key); 
        }    
    }    
} catch (IOException e) {    
    e.printStackTrace();   
}
```

**ByteBuffer使用**

创建ByteBuffer

```
//（1）使用allocate()静态方法
ByteBuffer buffer=ByteBuffer.allocate(256);
//以上方法将创建一个容量为256字节的ByteBuffer,如果发现创建的缓冲区容量太小,唯一的选择就是重新创建一个大小合适的缓冲区.

//（2）通过包装一个已有的数组来创建如下,通过包装的方法创建的缓冲区保留了被包装数组内保存的数据.
ByteBuffer buffer=ByteBuffer.wrap(byteArray);
// 如果要将一个字符串存入ByteBuffer,可以如下操作:
String sendString="你好,服务器. "; 
ByteBuffer sendBuffer=ByteBuffer.wrap(sendString.getBytes("UTF-16"));
```

缓冲区

```
buffer.flip();
//这个方法用来将缓冲区准备为数据传出状态,执行以上方法后,输出通道会从数据的开头而不是末尾开始.回绕保持缓冲区中的数据不变,只是准备写入而不是读取.
buffer.clear();
//这个方法实际上也不会改变缓冲区的数据,而只是简单的重置了缓冲区的主要索引值.不必为了每次读写都创建新的缓冲区,那样做会降低性能.相反,要重用现在的缓冲区,在再次读取之前要清除缓冲区.
```

3.3 一个简单例子
使用通道和ByteBuffer实现文件复制功能：

```
public void copy(String from, String to) throws IOException { 
    // 分配缓存 
    ByteBuffer buff = ByteBuffer.allocate(128); 
    // 输入、输出通道 
    FileChannel fin = null; 
    FileChannel fout = null; 
    try { 
        // 初始化输入输出通道 
        fin = new FileInputStream(from).getChannel(); 
        fout = new FileOutputStream(to).getChannel(); 
        // 从输入通道循环读取数据到缓存，并把缓存数据写入到输出通道 
        while (fin.read(buff) != -1) { 
            buff.flip(); 
            fout.write(buff); 
            buff.clear(); 
        } 
    } catch (FileNotFoundException e) { 
 
    } finally { 
        try { 
            if (fin != null) { 
                fin.close(); 
            } 
            if (fout != null) { 
                fout.close(); 
            } 
        } catch (IOException e) { 
            throw e; 
        } 
    } 
}
//如果需要将ByteBuffer转为FloatBuffer，则可以通过调用：
ByteBuffer buff = ByteBuffer.allocate(128); 
buff.asFloatBuffer()
//ByteBuffer转为其他的Buffer，如:CharBuffer、DoubleBuffer、IntBuffer、LongBuffer、ShortBuffer，都有对应的asXXXBuffer()方法。
```

### 反射机制

**什么是反射**

反射机制允许程序在运行时取得任何一个已知名称的class的内部信息，容许程序在运行时加载、探知、使用编译期间未知的class。即Java的反射机制可以加载一个运行时才得知名称的class，获得其完整结构。所谓的反射机制就是Java语言在运行时拥有一项自观的能力，即程序可以在运行时访问、检测和修改它本身状态或行为的一种能力。通过这种能力可以彻底的了解自身的情况为下一步的动作做准备。

**反射操作的对象**

在程序运行期间，Java运行时系统始终为所有的对象维护一个被称为运行时的类型标识。这个信息保存着每个对象所属的类足迹。虚拟机利用运行时信息选择相应的方法执行。然而，可以通过专门的Java类访问这些信息。保存这些信息的类称为Class，泛型形式为Class。Class是反射机制的基础，反射API通过操作Class来获取其完整结构。

Java的反射机制的实现要借助于4个类：`Class`，`Constructor`，`Field`，`Method`，通过这四个对象我们可以粗略的看到一个类的各个组成部分

**反射提供的功能**

Java 反射机制主要提供了以下功能：

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时调用任意一个对象的方法

两种编译：

- 静态编译：在编译时确定类型，绑定对象,即通过。
- 动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了Java的灵活性，体现了多态的应用，有以降低类之间的藕合性。

反射机制优缺点:

- 优点：就是可以实现动态创建对象和编译，体现出很大的灵活性，特别是在J2EE的开发中 它的灵活性就表现的十分明显。
- 缺点：对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于只直接执行相同的操作。并且它饶过了源码，会干扰原来的内部逻辑。

反射机制中常用API函数

```
// getClass()方式：
/** 
* Object类中的getClass()方法返回一个Class类型的实例 
*/ 
Boolean var1 = true; 
Class<?> classType1 = var1.getClass(); 
//输出：class java.lang.Boolean  
System.out.println(classType1);

//（2）T.class方式：
/** 
*  
* 运用T.class 语法(T是任意的Java类型) 
*/ 
Class<?> classType2 = Boolean.class; 
//输出：class java.lang.Boolean  
System.out.println(classType2);

//（3）Class.forName()方式：
/** 
*  
* 运用static method Class.forName()（使用时应该提供异常处理器） 
*/ 
Class<?> classType3 = Class.forName("java.lang.Boolean"); 
//输出：class java.lang.Boolean  
System.out.println(classType3);

//（4）TYPE语法方式：
/** 
*  
* 运用primitive wrapper classes的TYPE语法 
* 这里返回的是原生类型,和Boolean.class返回的不同 
*/ 
 
Class<?> classType4 = Boolean.TYPE; 
//输出：boolean  
System.out.println(classType4);
//注意：一个Class对象实际上表示的是一个类型，而这个类型未必一定是一种类。例如，int不是类，但int.class是一个Class类型的对象。虚拟机为每个类型管理一个Class对象。因此，可以用==运算符实现两个类对象比较的操作。



// Class常用的方法：
/** 
 * 返回类的名字 
 * 如：java.lang.String 
 */ 
String getName(); 
 
/** 
 * 快速地创建一个类的实例 
 * 调用默认构造器，如果该类没有默认构造器，抛出异常 
 * 如果要为构造器提供参数， 
 * 使用java.lang.reflect.Constructor中的newInstance方法 
 */ 
Object newInstance(); 
 
/** 
*  返回超类 
*/ 
getSuperclass(); 
 
/** 
* 给定名称的形式分别返回类支持的public域、方法和构造器数组， 
* 其中包括超类的公有成员 
*/ 
Field[] getFields(); 
Method[] getMethods(); 
Constructor<?>[] getConstructors(); 
 
/** 
* 获取指定的域、方法、构造函数 
*/ 
Field getField(String name) 
Method getMethod(String name, Class<?>... parameterTypes) 
Constructor<T> getConstructor(Class<?>... parameterTypes)
```

------

使用反射分析类

一个类主要由修饰符，域，构造器，方法组成，而Field、Method、Constructor类，分别用于描述类的域、方法和构造器。另外java.lang.reflect包中的Modifier类可以分析访问修饰符。那么用它们就可以分析类。

- Class getDeclaringClass() 返回一个用于描述类中定义的构造器、方法或域的Class对象
- String getName() 返回相应条目的名称
- int getModifiers() 返回整型数值，用不同的位开关描述访问修饰符的使用状况
- Constructor Class[] getExceptionTypes() 返回一个用于描述方法抛出的异常类型的Class对象数组
- Class[] getParameterTypes() 返回一个用于描述参数类型的Class对象数组
- Field Class getType() 用于返回描述域所属类型的Class类型对象
- static String toString(int modifiers) 返回对应modifiers位设置的修饰符的字符串表示
- static boolean isXXX(int modifiers) 检测方法名中对应的修饰符在modifiers中的值

访问权限问题：

由于反射机制的默认行为受限于Java的访问控制，比如，访问私有的方法，字段，除非拥有访问权限，否则Java安全机制允许查看任意对象有哪些域，而不允许读它们的值（读取将抛异常）。然而如果一个Java程序没有受到安全管理器的控制，就可以覆盖访问控制。为了达到这个目的，就需要调用Field、Method、Constructor对象的`setAccessible()`方法。

- void setAccessible(boolean flag) 为反射对象设置可访问标志，flag为true表明屏蔽Java语言的访问检查，使得对象的私有属性也可以被查询和设置
- boolean isAccessible() 返回反射对象的可访问标志的值
- static void setAccessible(AccessibleObject[] array, boolean flag) 一种设置对象数组可访问标志的快捷方法

### Object的公用方法

Object的共有方法如下：

```
//创建并返回此对象的一个副本。 
protected  Object    clone() ; 
 
//指示其他某个对象是否与此对象“相等” 
boolean    equals(Object obj) ; 
 
/** 
* 当垃圾回收器确定不存在对该对象的更多引用时， 
* 由对象的垃圾回收器调用此方法 
*/ 
protected void finalize() ; 
 
//返回此 Object 的运行时类 
Class getClass(); 
 
//返回该对象的哈希码值 
int hashCode(); 
 
//唤醒在此对象监视器上等待的单个线程 
void notify(); 
 
//唤醒在此对象监视器上等待的所有线程 
void notifyAll(); 
 
//返回该对象的字符串表示 
String toString(); 
 
/** 
* 在其他线程调用此对象的 notify() 
* 方法或 notifyAll() 方法前，导致当前线程等待 
*/ 
void wait(); 
 
/** 
* 在其他线程调用此对象的 notify() 方法或 notifyAll() 方法， 
* 或者超过指定的时间量前，导致当前线程等待 
*/ 
void wait(long timeout); 
 
/** 
* 在其他线程调用此对象的 notify() 方法或 notifyAll() 方法， 
* 或者其他某个线程中断当前线程，或者已超过某个实际时间量前， 
* 导致当前线程等待 
*/ 
void wait(long timeout, int nanos);
```

### wait()和sleep()的区别

**父类方面**

这两个方法来自不同的类分别是：`sleep`来自`Thread`类；`wait`来自`Object`类。

sleep是Thread的静态类方法，谁调用的谁去睡觉，即使在a线程里调用b的sleep方法，实际上还是a去睡觉，要让b线程睡觉要在b的代码中调用sleep。

**锁方面**

最主要是`sleep`方法没有释放锁，而`wait`方法释放了锁，使得其他线程可以使用同步控制块或者方法。
`sleep`不出让系统资源；`wait`是进入线程等待池等待，出让系统资源，其他线程可以占用CPU。一般`wait`不会加时间限制，因为如果`wait`线程的运行资源不够，再出来也没用，要等待其他线程调用`notify/notifyAll`唤醒等待池中的所有线程，才会进入就绪队列等待OS分配系统资源。`sleep(milliseconds)`可以用时间指定使它自动唤醒过来，如果时间不到只能调用`interrupt()`强行打断。
`Thread.sleep(0)`的作用是“触发操作系统立刻重新进行一次CPU竞争”。

**使用范围方面**

wait，notify和notifyAll只能在同步控制方法或者同步控制块里面使用，而sleep可以在任何地方使用。
如：

```
synchronized(x){  
      x.notify()  
     //或者wait()  
}
```

### BlockingQueue CountDownLatch

`BlockingQueue`

`BlockingQueue`接口定义了一种阻塞的`FIFO queue`，每一个`BlockingQueue`都有一个容量：
当容量满时，往`BlockingQueue`中添加数据时会造成阻塞,当容量为空时，取元素操作会阻塞。
`BlockingQueue`有两个实现类：`ArrayBlockingQueue`和`LinkedBlockingQueue`。

`ArrayBlockingQueue`

- 一个由数组支持的有界阻塞队列
- 按先进先出原则排序
- 一旦创建好这个数组，就不能再增加其容量
- 试图向已满的队列中放入元素会导致操作受阻塞
- 试图从空的队列中提取元素将导致类似的阻塞。

`LinkedBlockingQueue`

- LinkedBlockingQueue是一个基于已链接节点的、范围任意的blocking queue的实现
- 此队列按FIFO（先进先出）排序元素。队列的头部 是在队列中时间最长的元素。队列的尾部 是在队列中时间最短的元素
- 新元素插入到队列的尾部，并且队列检索操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低.
- 可选的容量范围构造方法参数作为防止队列过度扩展的一种方法。
- 如果未指定容量，则它等于Integer.MAX_VALUE。除非插入节点会使队列超出容量，否则每次插入后会动态地创建链接节点 ，容量范围可以在构造方法参数中指定作为防止队列过度扩展。
- 此对象是 线程阻塞-安全的
- 不接受null元素
- 实现了Collection和Iterator接口的所有可选 方法
- 在JDK5/6中，LinkedBlockingQueue和ArrayBlocingQueue等对象的poll(long timeout, TimeUnit unit)存在内存泄露Leak的对象AbstractQueuedSynchronizer.Node，据称JDK5会在Update12里Fix，JDK6会在Update2里Fix

**ArrayBlockingQueue和LinkedBlockingQueue的区别**

- 队列中锁的实现不同
  `ArrayBlockingQueue`实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁；
  `LinkedBlockingQueue`实现的队列中的锁是分离的，即生产用的是putLock，消费是takeLock
  在生产或消费时操作不同
- `ArrayBlockingQueue`实现的队列中在生产和消费的时候，是直接将枚举对象插入或移除的；
  `LinkedBlockingQueue`实现的队列中在生产和消费的时候，需要把枚举对象转换为Node进行插入或移除，会影响性能
- 队列大小初始化方式不同
  `ArrayBlockingQueue`实现的队列中必须指定队列的大小；
  `LinkedBlockingQueue`实现的队列中可以不指定队列的大小，但是默认是`Integer.MAX_VALUE`

------

`CountDownLatch`

一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。主要方法

```
public CountDownLatch(count); 
public void countDown(); 
public void await() ;
```

构造方法指定了计数的次数。countDown方法，当前线程调用此方法则计数减一。await方法，调用该方法会一直阻塞当前线程，直到计时器的值为0.

### 锁的等级

`synchronized`在修饰代码块的时候需要一个`reference`对象作为锁的对象. 在修饰实例方法的时候默认是当前实例对象作为锁的对象. 在修饰类方法（静态方法）时候默认是当前类的`Class`对象作为锁的对象.

`synchronized`使用总结如下

- 对象锁钥匙只能有一把才能互斥，才能保证共享变量的唯一性
- 在静态方法上的锁，和实例方法上的锁，默认不是同样的，如果同步需要制定两把锁一样。
- 关于同一个类的方法上的锁，来自于调用该方法的对象，如果调用该方法的对象是相同的，那么锁必然相同，否则就不相同。比如 new A().x() 和 new A().x(),对象不同，锁不同，如果A的单例的，就能互斥。
- 静态方法加锁，能和所有其他静态方法加锁的 进行互斥
- 静态方法加锁，和xx.class 锁效果一样，直接属于类的

### synchronized lock reentrantLock

**synchronized**

当它用来修饰一个方法或者一个代码块的时候，能够保证在同一时刻最多只有一个线程执行该段代码，它是在 软件层面依赖JVM实现同步。

`synchronized 方法的缺陷：`

> 若将一个大的方法声明为synchronized将会大大影响效率，典型地，若将线程类的方法run()声明为synchronized，由于在线程的整个生命期内它一直在运行，因此将导致它对本类任何synchronized方法的调用都永远不会成功。

解决方案: 通过 synchronized关键字来声明synchronized 块。

------

**Lock**

`Lock`接口实现提供了比使用`synchronized`方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的`Condition`对象。在硬件层面依赖特殊的CPU指令实现同步更加灵活。

**什么是Condition**

`Condition`接口将`Object`监视器方法（`wait`、`notify` 和 `notifyAll`）分解成截然不同的对象，以便通过将这些对象与任意 Lock实现组合使用，为每个对象提供多个等待`set`（wait-set）。其中，`Lock`替代了`synchronized`方法和语句的使用，`Condition`替代了 `Object`监视器方法的使用。

虽然`synchronized`方法和语句的范围机制使得使用监视器锁编程方便了很多，而且还帮助避免了很多涉及到锁的常见编程错误，但有时也需要以更为灵活的方式使用锁。例如，某些遍历并发访问的数据结果的算法要求使用”hand-over-hand”或”chain locking”：获取节点 A的锁，然后再获取节点B的锁，然后释放A并获取C，然后释放B并获取D，依此类推。Lock接口的实现允许锁在不同的作用范围内获取和释放，并允许以任何顺序获取和释放多个锁，从而支持使用这种技术。

随着灵活性的增加，也带来了更多的责任。不使用块结构锁就失去了使用`synchronized`方法和语句时会出现的锁自动释放功能。在大多数情况下，应该使用以下语句：

```
Lock l = ...; //lock接口的实现类对象 
l.lock(); 
try { 
    // access the resource protected by this lock 
} finally { 
    l.unlock(); 
}
```

在`java.util.concurrent.locks`包中有很多`Lock`的实现类，常用的有`ReentrantLock`、`ReadWriteLock`（实现类ReentrantReadWriteLock）.它们是具体实现类，不是Java语言关键字。

------

**ReentrantLock**

一个可重入的互斥锁**Lock**，它具有与使用`synchronized`方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大。

最典型的代码如下:

```
class X { 
   private final ReentrantLock lock = new ReentrantLock(); 
   // ... 
 
   public void m() {  
     lock.lock();  // block until condition holds 
     try { 
       // ... method body 
     } finally { 
       lock.unlock() 
     } 
   } 
 }
```

> 重入性：指的是同一个线程多次试图获取它所占有的锁，请求会成功。当释放锁的时候，直到重入次数清零，锁才释放完毕。

`ReentrantLock`的`lock`机制有2种，**忽略中断锁**和**响应中断锁**，这给我们带来了很大的灵活性。比如：如果A、B 2个线程去竞争锁，A线程得到了锁，B线程等待，但是A线程这个时候实在有太多事情要处理，就是一直不返回，B线程可能就会等不及了，想中断自己，不再等待这个锁了，转而处理其他事情。这个时候`ReentrantLock`就提供了2种机制，第一，B线程中断自己（或者别的线程中断它），但是ReentrantLock不去响应，继续让B线程等待，你再怎么中断，我全当耳边风（`synchronized`原语就是如此）；第二，B线程中断自己（或者别的线程中断它），`ReentrantLock` 处理了这个中断，并且不再等待这个锁的到来，完全放弃。

**ReentrantLock相对于synchronized多了三个高级功能**

1. 等待可中断: 在持有锁的线程长时间不释放锁的时候,等待的线程可以选择放弃等待.`tryLock(long timeout, TimeUnit unit)`
2. 公平锁: 按照申请锁的顺序来依次获得锁称为公平锁.`synchronized`的是非公平锁,`ReentrantLock`可以通过构造函数实现公平锁.`new RenentrantLock(boolean fair)`公平锁和非公平锁。这2种机制的意思从字面上也能了解个大概：即对于多线程来说，公平锁会依赖线程进来的顺序，后进来的线程后获得锁。而非公平锁的意思就是后进来的锁也可以和前边等待锁的线程同时竞争锁资源。对于效率来讲，当然是非公平锁效率更高，因为公平锁还要判断是不是线程队列的第一个才会让线程获得锁。
3. 绑定多个`Condition`: 通过多次`newCondition`可以获得多个Condition对象,可以简单的实现比较复杂的线程同步的功能.通过`await()`，`signal()`

------

**synchronized和lock的用法与区别**

1. `synchronized`是托管给JVM执行的，而`Lock`是`Java`写的控制锁的代码。
2. `synchronized`原始采用的是CPU悲观锁机制，即线程获得的是独占锁。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。　
3. `Lock`用的是乐观锁方式。每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。
4. `ReentrantLock`必须在`finally`中释放锁，否则后果很严重，编码角度来说使用`synchronized`更加简单，不容易遗漏或者出错。
5. `ReentrantLock`提供了可轮询的锁请求，他可以尝试的去取得锁，如果取得成功则继续处理，取得不成功，可以等下次运行的时候处理，所以不容易产生死锁，而`synchronized`一旦进入锁请求要么成功，要么一直阻塞，所以更容易产生死锁。
6. `synchronized`的话，锁的范围是整个方法或`synchronized`块部分；而`Lock`因为是方法调用，可以跨方法，灵活性更大

一般情况下都是用`synchronized`原语实现同步，除非下列情况使用`ReentrantLock`:

- 某个线程在等待一个锁的控制权的这段时间需要中断
- 需要分开处理一些wait-notify，`ReentrantLock`里面的`Condition`应用，能够控制`notify`哪个线程
- 具有公平锁功能，每个到来的线程都将排队等候

### 线程池

**线程池基础**

配置线程池一般如下语句：

```
public static final Executor THREAD_POOL_EXECUTOR = new ThreadPoolExecutor( 
                    CORE_POOL_SIZE,MAXIMUM_POOL_SIZE,KEEP_ALIVE,TimeUnit.SECONDS, 
                      sPoolWorkQueue,sThreadFactory 
                      );
```

当一个任务加入到线程池时:

1. 如果此时线程池中的数量小于`corePoolSize`，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。
2. 如果此时线程池中的数量等于 `corePoolSize`，但是缓冲队列 `workQueue`未满，那么任务被放入缓冲队列。
3. 如果此时线程池中的数量大于`corePoolSize`，缓冲队列`workQueue`满，并且线程池中的数量小于`maximumPoolSize`，新建线程来处理被添加的任务。
4. 如果此时线程池中的数量大于`corePoolSize`，缓冲队列`workQueue`满，并且线程池中的数量等于`maximumPoolSize`，那么通过 `handler`所指定的策略来处理此任务。
5. 当线程池中的线程数量大于 `corePoolSize`时，如果某线程（非核心线程）空闲时间超过`keepAliveTime`，线程将被终止。这样，线程池可以动态的调整池中的线程数。

也就是,处理任务的优先级为：

- 核心线程corePoolSize
- 任务队列workQueue
- 最大线程maximumPoolSize

如果三者都满了，使用handler处理被拒绝的任务（一般为抛出java.util.concurrent.RejectedExecutionException异常）

------

**线程池类型**

线程池主要有以下4种：

1. `FixedThreadPool`：线程数量固定的线程池，线程处于空闲状态时不会被回收，除非线程被关闭。当所有线程都处于活动状态时，新的任务都会处于等待状态，直到有线程空闲出来。
2. `CachedThreadPool`：线程数量不固定，只有非核心线程，可以放任意多个线程（Integer.MAX_VALUE）,线程池里所有线程处于活动状态时，创建新的线程处理新来的任务。否则利用闲置的线程处理新任务。线程池里空闲线程有超时机制，时长为60秒。
3. `ScheduledThreadPool`：核心线程数量是固定的，非核心线程是没有限制。当非核心线程闲置时会被立即回收。
4. `SingleThreadExector`：内部只有一个核心线程，确保所有任务在同一个线程中按顺序执行。

------

**线程池使用方法**

```
Runnable task=new Runnable(){ 
     Public void run(){ 
       //TODO ....... 
    } 
}; 
 
//FixedThreadPool使用 
ExecutorService fixedThreadPool=Executors.newFixedThreadPool(4); 
fixedThreadPool.execute(task); 
 
//CachedThreadPool的使用 
ExecutorService cachedThreadPool=Executors.newCachedThreadPool(); 
cachedThreadPool.execute(task);  
 
//ScheduledThreadPool的使用 
ExecutorService scheduledThreadPool=Executors.newScheduledThreadPool(4); 
//2000ms后执行task 
scheduledThreadPool.schedule(task,2000,TimeUnit.MILLISECONDS); 
//延迟10ms后，每隔1000ms执行一次task 
scheduledTheadPool.scheduleAtFixedRate(task,10,1000,TimeUnit.MILLISECONDS);  
 
//SingleThreadExector的使用 
ExecutorService sigleThreadPool=Executors.newSingleThreadExecutor(); 
fixedThreadPool.execute(task);
```

------

**线程池的优点**

- 重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销
- 能有效控制线程池的最大并发数，避免大量线程之间因互相抢占系统资源而导致阻塞。
- 能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能。

### 线程 ThreadLocal

创建新线程的两种方法:

- 通过实现`Runable`接口,再将`Runnable`作为`Thread`的构造函数参数创建Thread对象
- 通过继承Thread类本身。

其实，Java中真正能创建新线程的只有Thread对象，通过Runnable的方式，最终还是需要通过Thread对象来创建线程。

当`JVM`启动时，通常都会有单个非守护线程（它通常会调用某个指定类的main方法）。`JVM`会继续执行线程，直到下列任一情况出现时为止：

- 调用了Runtime类的`exit`方法，并且安全管理器允许退出操作发生。
- 非守护线程的所有线程都已停止运行，无论是通过从对run方法的调用中返回，还是通过抛出一个传播到run方法之外的异常。

------

**ThreadLocal设计理念和作用**

**设计理念**

为每个线程创造一个资源的复本。将每一个线程存取数据的行为加以隔离，实现的方法就是给予每个线程一个特定空间来保管该线程所独享的资源。

**作用**

为每一个使用该变量的线程都提供一个变量值的副本，是每一个线程都可以独立地改变自己的副本，而不会和其它线程的副本冲突。从线程的角度看，就好像每一个线程都完全拥有该变量。

**ThreadLocal使用**

`ThreadLocal`实例通常是类中的`private static`字段，它们希望将状态与某一个线程相关联。每个线程都保持对其线程局部变量副本的隐式引用，只要线程是活动的并且`ThreadLocal`实例是可访问的，在线程消失之后，其线程局部实例的所有副本都会被垃圾回收（除非存在对这些副本的其他引用）。

```
// 首先创建ThreadLocal对象：
ThreadLocal<Integer> mValue=new ThreadLocal<Integer>();
// 然后在线程中调用set和get方法来设置和获取值，例如：
mValue.set(1); 
int value=mValue.get();
```

**实现原理**

简单地说，就是在`ThreadLocal`类中有一个`Map`，用于存储每一个线程的变量的副本。`Map`的`Key`是`Thread`，`value`就是副本的值。

深入源码去看，`ThreadLocal`把线程和线程局部变量存在`ThreadLocalMap`中，而`ThreadLocalMap`是`ThreadLocal`的静态内部类，我们来看看`ThreadLocalMap`的部分源码：

```
static class Entry extends WeakReference<ThreadLocal<?>> { 
 
    Object value; 
    Entry(ThreadLocal<?> k, Object v) { 
        super(k); 
        value = v; 
    } 
}
```

这个`Map`的`key`是`ThreadLocal`对象的弱引用，当要抛弃掉`ThreadLocal`对象时，垃圾收集器会忽略这个`key`的引用而清理掉`ThreadLocal`对象 。

那么到底是`ThreadLocal`还是`Thread`持有`ThreadLocalMap`对象的引用呢？

```
/* ThreadLocal values pertaining to this thread.  
 * This map is maintained by the ThreadLocal class. 
 */ 
ThreadLocal.ThreadLocalMap threadLocals = null;
```

`ThreadLocalMap`变量属于`Thread`的内部属性,不同的`Thread`拥有完全不同的`ThreadLocalMap`变量.`Thread`中的`ThreadLocalMap`变量的值是在`ThreadLocal`对象进行set或者get操作时创建的.　 　　 在创建`ThreadLocalMap`之前,会首先检查当前`Thread`中的`ThreadLocalMap`变量是否已经存在,如果不存在则创建一个；如果已经存在,则使用当前`Thread`已创建的`ThreadLocalMap`.　

**ThreadLocal的接口方法**

```
/** 
 * 返回此线程局部变量的初始值 
 * 
 * 线程第一次使用 get() 方法访问变量时将调用此方法，但如果线程之前调用 
 * 了 set(T) 方法，则不会对该线程再调用 initialValue 方法。通常，此 
 * 方法对每个线程最多调用一次，但如果在调用 get() 后又调用了  
 * remove()，则可能再次调用此方法。 
 *  
 * 该实现返回 null；如果程序员希望线程局部变量具有 null 以外的值，则 
 * 必须为 ThreadLocal 创建子类，并重写此方法。通常将使用匿名内部类完 
 * 成此操作。 
*/ 
protected T initialValue(); 
 
/** 
* 返回此线程局部变量的当前线程的值 
*  
* 如果变量没有用于当前线程的值，则先 
* 将其初始化为调用initialValue() 方法返回的值。 
*/ 
public T get(); 
 
/** 
* 将此线程局部变量的当前线程副本中的值设置为指定值 
*  
* 大部分子类不需要重写此方法，它们只依靠 initialValue() 方法 
* 来设置线程局部变量的值 
*/ 
public void set(T value); 
 
/** 
* 移除此线程局部变量当前线程的值 
*  
* 如果此线程局部变量随后被当前线程 读取，且这期间当前线程没有 
* 设置其值，则将调用其 initialValue() 方法重新初始化其值。 
* 这将导致在当前线程多次调用 initialValue 方法。 
*/ 
public void remove();
```

如果希望线程局部变量初始化其它值，那么需要自己实现ThreadLocal的子类并重写该方法，通常使用一个内部类对ThreadLocal进行实例化。

**ThreadLocal如何做到线程安全**

从上面的分析我们可以得出：

- 因为每个`Thread`在进行对象访问时,访问的都是各自线程自己的`ThreadLocalMap`，所以保证了`Thread`与`Thread`之间的数据访问隔离。
- 不同的`ThreadLocal`实例操作同一`Thread`时，`ThreadLocalMap`在存储时采用当前`ThreadLocal`的实例作为`key`来保证数据访问隔离（上面源码Entry处可以看出）。　

**TheadLocal模式与同步机制的区别**

1. 实现机制: 同步机制采用了“以时间换空间”的方式,提供一份变量,让不同的线程排队访问.而ThreadLocal采用了“以空间换时间”的方式,为每一个线程都提供一份变量的副本,从而实现同时访问而互不影响。
2. 同步共享方面: Java中的synchronized是一个保留字,它依靠JVM的锁机制来实现临界区的函数或者变量的访问中的原子性.在同步机制中,通过对象的锁机制保证同一时间只有一个线程访问变量.此时,被用作“锁机制”的变量是多个线程共享的；而ThreadLocal会为每一个线程维护一个和该线程绑定的变量的副本，从而隔离了多个线程的数据，每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。　 　 3.使用场合同步机制是为了同步多个线程对相同资源的并发访问，是为了多个线程之间进行通信的有效方式。而ThreadLocal是隔离多个线程的数据共享，从根本上就不在多个线程之间共享资源（变量），这样当然不需要对多个线程进行同步了。所以，如果你需要进行多个线程之间进行通信，则使用同步机制。如果需要隔离多个线程之间的共享冲突，可以使用ThreadLocal。

### Java的四种引用

> 从JDK1.2版本开始，把对象的引用分为四种级别，从而使程序能更加灵活的控制对象的生命周期。这四种级别由高到低依次为：强引用、软引用、弱引用和虚引用。

**强引用(StrongReference)**

强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，JVM宁愿抛出`OutOfMemoryError`错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。如代码String s=”abc”中变量s就是字符串对象”abc”的一个强引用。只要你给强引用对象s赋空值null,该对象就可以被垃圾回收器回收。因为该对象此时不再含有其他强引用。

**软引用（SoftReference）**

如果内存足够，不回收；如果内存不足，则回收。软引用可用来实现内存敏感的高速缓存。软引用可以和引用队列`ReferenceQueue`联合使用，如果软引用的对象被垃圾回收，JVM就会把这个软引用加入到与之关联的引用队列中。

例如:

```
String str=new String("Test");
ReferenceQueue<String> rq=new ReferenceQueue<String>(); 
SoftReference<String> sr=new SoftReference<String>(str,rq); 
str = null;         // 将强引用撤销

// 或者
SoftReference<String> sr=new SoftReference<String>(str); 
str=null;//将强引用撤掉

// 取出对象
String s = sr.get();
```

如果被回收，则s为null，否则，s即为str所指引的对象”Test”

**弱引用(WeakReference)**

弱引用与软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管辖的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象，弱引用非常适合存储元数据。另一个使用弱引用的例子是`WeakHashMap`，它是除`HashMap`和`TreeMap`之外，Map接口的另一种实现。`WeakHashMap`有一个特点：`Map`中的键值(keys)都被封装成弱引用，也就是说一旦强引用被删除，`WeakHashMap`内部的弱引用就无法阻止该对象被垃圾回收器回收。弱引用的使用跟软引用使用方式相同，只是将`SoftReference`替换为`WeakReference`。

**虚引用(PhantomReference)**

“虚引用”顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。 虚引用主要用来跟踪对象被垃圾回收器回收的活动。虚引用与软引用和弱引用的一个区别在于：虚引用 必须 和引用队列 （`ReferenceQueue`）联合使用。当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

### static关键字

**static静态修饰符**

在程序中任何变量或者代码都是在编译时由系统自动分配内存来存储的。`static`修饰符表示静态的，在类加载时`JVM`会把它放到**方法区**，被本类以及本类的所有实例所共用。在编译后所分配的内存会一直存在，直到程序退出内存才会释放这个空间。如果一个被所有实例共用的方法被申明为`static`，那么就可以节省空间，不用每个实例初始化的时候都被分配到内存。

我们比较常见的`static`修饰是在静态变量和静态方法中。它们可以直接通过类名来访问。下面我们看看关于静态代码块和静态类。

------

**静态代码块**

说起静态代码块，就不得不说类初始化。类初始化是类加载的最后一步，前面类加载过程中，除了加载阶段用户可以通过自定义类加载器参与以外，其余动作都是虚拟机主导和控制。到了初始化阶段，才是真正执行类中定义Java程序代码。

准备阶段中，变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划初始化类变量。初始化过程其实是执行类构造器`<clinit>()`方法的过程。

`<clinit>()`方法是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的。收集的顺序是按照语句在源文件中出现的顺序。静态语句块中只能访问定义在静态语句块之前的变量，定义在它之后的变量可以赋值，但不能访问。

`<clinit>()`方法与类构造函数（或者说实例构造器()）不同，他不需要显式地调用父类构造器，虚拟机会保证子类的`<clinit>()`方法执行之前，父类的`<clinit>()`已经执行完毕。

------

**静态类**

- 只能在内部类中定义静态类
- 静态内部类与外层类绑定，即使没有创建外层类的对象，它一样存在。
- 静态类的方法可以是静态的方法也可以是非静态的方法，静态的方法可以在外层通过静态类调用，而非静态的方法必须要创建类的对象之后才能调用。
- 只能引用外部类的static成员变量（也就是类变量）。
- 如果一个内部类不是被定义成静态内部类，那么在定义成员变量或者成员方法的时候，是不能够被定义成静态的。

### 集合的区别

**Map**

> 键映射到值的对象。

有如下特点:

- 一个映射不能包含重复的键
- 每个键最多只能映射到一个值。
- 某些映射实现可明确保证其顺序，如TreeMap类
- 另一类映射实现则不保证顺序如HashMap

`Map`可以将`key`序列、`value`序列单独抽取出来。使用`keySet()`抽取key序列，将所有`key`生成一个`Set`。使用`values`抽取`value`序列，将所有value生成一个`Collection`，为什么key生成`Set`，而value生成`Collection`？因为key总是独一无二，`value`允许重复。

Map接口的部分函数原型如下:

```
Set<K> keySet(); 
Collection<V> values(); 
 
V remove(Object key); 
V get(Object key); 
V put(K key, V value); 
void putAll(Map<? extends K, ? extends V> m); 
 
boolean containsKey(Object key); 
boolean containsValue(Object value); 
 
void clear(); 
int size(); 
boolean isEmpty();
```

**Set**

不能包含重复元素的`Collection`。如下特征：

- Set不包含满足e1.euqals(e2)
- 最多包含一个null元素(这里是指HashSet,TreeSet不支持插入null)
- 不可随机访问包含的元素
- Set没有同步方法。

Set接口的部分函数原型如下:

```
boolean add(E e); 
boolean remove(Object o); 
 
boolean addAll(Collection<? extends E> c); 
boolean removeAll(Collection<?> c); 
 
boolean contains(Object o); 
boolean containsAll(Collection<?> c); 
 
boolean retainAll(Collection<?> c); 
 
Object[] toArray(); 
T[] toArray(T[] a); 
 
void clear(); 
int size(); 
boolean isEmpty();
```

------

**List**

如下特征：

- 可随机访问包含的元素
- 元素是有序的
- 可在任意位置增、删元素
- 允许重复元素。

List接口的部分函数原型如下:

```
E get(int index); 
E set(int index, E element); 
 
boolean add(E e); 
boolean remove(Object o); 
 
boolean addAll(Collection<? extends E> c); 
boolean removeAll(Collection<?> c); 
 
boolean contains(Object o); 
boolean containsAll(Collection<?> c); 
 
int indexOf(Object o); 
int lastIndexOf(Object o); 
 
boolean retainAll(Collection<?> c); 
 
Object[] toArray(); 
 
void clear(); 
int size(); 
boolean isEmpty();
```

**Queue**

队列，特点是先进先出。

`Queue`在使用时尽量避免`Collection`的`add()`和`remove()`方法，而是要使用`offer()`来加入元素，使用`poll`来获取并移出元素。他们的优点是通过返回值可以判断成功与否。`add()`和`remove()`方法在失败的时候会抛出异常。

如果使用而不移出该元素，使用`element()`或者`peek()`方法。值得注意的是`LinkedList`类实现了`Queue`接口，因此我们可以把`LinkedList`当初`Queue`来用。

`Queue`实现通常不允许插入`null`元素。尽管某些实现（如LinkedList）并不禁止将null插入到Queue中，即使在允许null的实现中，也不应将null插入到Queue中，因为`null`也作`poll`方法的一个特殊返回值，表明队列不包含元素。

Queue接口的部分函数原型如下:

```
//插入新元素到队列，如果插入成功，返回true， 
//如果队列已满，抛出IllegalStateException异常 
boolean add(E e); 
 
//插入新元素到队列，如果插入成功返回true 
//如果队列已满，返回false，但不抛出异常 
boolean offer(E e); 
 
//返回第一个元素，并将该元素从队列中删除 
//如果队列为空，抛出异常 
E remove(); 
 
//返回第一个元素，并将该元素从队列中删除 
//如果队列为空，返回null 
E poll(); 
 
//返回队列的第一个元素， 
//如果队列为空，抛异常 
E element(); 
 
//返回队列的第一个元素， 
//如果队列为空，返回null 
E peek();
```

`Queue`接口有子接口`BlockingQueue`和`Deque`，`BlockingQueue`表示阻塞队列；`Deque`是双向队列，即可以从两端插入元素。`BlockingQueue`有如下几个实现类：

- ArrayBlockingQueue
- LinkedBlockingQueue

主要看看BlockingQueue的两个新方法：

```
//插入一个新元素，如果队列已满，则一直等待（阻塞） 
void put(E e) throws InterruptedException; 
 
//返回队列的第一个元素并将该元素从队列里删除，如果队里为空，则一直等待（阻塞） 
E take() throws InterruptedException;
```

`ArrayBlockingQueue`和`LinkedBlockingQueue`是阻塞队列的不同实现，即一个是通过数组方式，一个是通过链表的方式实现的阻塞队列。

有个特殊的接口`BlockingDeque`，`BlockingDeque`既实现了`BlockingQueue`又实现了`Deque`,而`BlockingDeque`的实现类有

- LinkedBlockingDeque
- 前面我们说到，接口Deque是双向队列。接口Deque添加了如下新方法：

```
void addFirst(E e); 
void addLast(E e); 
boolean offerFirst(E e); 
boolean offerLast(E e); 
E removeFirst(); 
E removeLast(); 
E pollFirst(); 
E pollLast(); 
E getFirst(); 
E getLast(); 
E peekFirst(); 
E peekLast();
```

------

**Stack**

`Stack`继承自`Vector`（可增长的对象数组），也是同步的。他通过五个操作对类Vector进行了扩展，允许将向量视为堆栈。他提供了通常的`push`和`pop`操作，以及取堆栈顶点的`peek`方法。测试堆栈是否为空的`empty`方法、在堆栈中查找项并确定对堆栈顶距离的`search`方法。

Stack类的部分函数原型如下:

```
public E push(E item); 
public synchronized E pop(); 
 
//返回栈顶的元素，但不将其出栈 
public synchronized E peek(); 
public synchronized int search(Object o); 
public boolean empty();
```

### 异常

异常类继承关系图:

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/throwable.png)

**Throwable**

`Throwable`类是所有错误或异常的超类。只有当对象是此类或其子类之一的实例时，才能通过`JVM`或者是通过`throw`语句抛出；另外`catch`子句中的参数类型也必须是该类型。

`Throwable`类及其子类有两个构造方法:

- 不带参数
- 带有 String 参数，此参数可用于生成详细消息。

`Throwable`包含了其线程创建时线程执行堆栈的快照。它还包含了给出有关错误更多信息的消息字符串。`Java`将可抛出(Throwable)的结构分为三种类型：

- 错误(Error)
- 运行时异常(RuntimeException)
- 被检查的异常(Checked Exception)

------

**Error**

`Error`是`Throwable`的子类，用于指示合理的应用程序不应该试图捕获的严重问题。大多数这样的错误都是异常条件。和`RuntimeException`一样， 编译器也不会检查`Error`。当资源不足、约束失败、或是其它程序无法继续运行的条件发生时，就产生错误，程序本身无法修复这些错误的。 　

------

**Exception**

`Exception`类及其子类是`Throwable`的一种形式，它指出了合理的应用程序想要捕获的条件。对于可以恢复的条件使用被检查异常（Exception的子类中除了`RuntimeException`之外的其它子类），对于程序错误使用运行时异常。　

`ClassNotFoundException`

当应用程序试图使用以下方法通过字符串名加载类时：

- Class 类中的 forName 方法。
- ClassLoader 类中的 findSystemClass 方法。
- ClassLoader 类中的 loadClass 方法。

但是没有找到具有指定名称的类的定义，抛出该异常。

`CloneNotSupportedException`

当调用`Object`类中的`clone`方法复制对象，但该对象的类无法实现`Cloneable`接口时，抛出该异常。重写`clone`方法的应用程序也可能抛出此异常，指示不能或不应复制一个对象。

`IOException`

当发生某种I/O异常时，抛出此异常。此类是失败或中断的I/O操作生成的异常的通用类。

- EOFException: 当输入过程中意外到达文件或流的末尾时，抛出此异常。此异常主要被数据输入流用来表明到达流的末尾。注意：其他许多输入操作返回一个特殊值表示到达流的末尾，而不是抛出异常。
- FileNotFoundException: 当试图打开指定路径名表示的文件失败时，抛出此异常。在不存在具有指定路径名的文件时，此异常将由 `FileInputStream`、`FileOutputStream`和`RandomAccessFile`构造方法抛出。如果该文件存在，但是由于某些原因不可访问，比如试图打开一个只读文件进行写入，则此时这些构造方法仍然会抛出该异常。
- MalformedURLException: 抛出这一异常指示出现了错误的URL。或者在规范字符串中找不到任何合法协议，或者无法解析字符串。　
- UnknownHostException: 指示主机IP地址无法确定而抛出的异常。

------

**RuntimeException**

是那些可能在Java虚拟机正常运行期间抛出的异常的超类。可能在执行方法期间抛出但未被捕获的`RuntimeException`的任何子类都无需在`throws`子句中进行声明。`Java`编译器不会检查它。当程序中可能出现这类异常时，还是会编译通过。虽然`Java`编译器不会检查运行时异常，但是我们也可以通过`throws`进行声明抛出，也可以通过`try-catch`对它进行捕获处理。

- `ArithmeticException`：当出现异常的运算条件时，抛出此异常。例如，一个整数“除以零”时，抛出此类的一个实例。

- `ClassCastException`：当试图将对象强制转换为不是实例的子类时，抛出该异常。例如：Object x = new Integer(0);

- `IllegalArgumentException`：抛出的异常表明向方法传递了一个不合法或不正确的参数。

- `IllegalStateException`：在非法或不适当的时间调用方法时产生的信号。换句话说，即Java环境或Java应用程序没有处于请求操作所要求的适当状态下。

- `IndexOutOfBoundsException`：指示某排序索引（例如对数组、字符串或向量的排序）超出范围时抛出。 应用程序可以为这个类创建子类，以指示类似的异常。

- `NoSuchElementException`：由`Enumeration`的`nextElement`方法抛出，表明枚举中没有更多的元素。

- ```
  NullPointerException
  ```

  ：当应用程序试图在需要对象的地方使用null时，抛出该异常。这种情况包括：

  1. 调用null对象的实例方法。
  2. 访问或修改null对象的字段。
  3. 将null作为一个数组，获得其长度。
  4. 将null作为一个数组，访问或修改其时间片。
  5. 将null作为Throwable值抛出。
  6. 应用程序应该抛出该类的实例，指示其他对null对象的非法使用。

------

**SOF（堆栈溢出 StackOverflow）**

> 当应用程序递归太深而发生堆栈溢出时，抛出StackOverflowError错误。

程序中一旦出现死循环或者是大量的递归调用，在不断的压栈过程中，造成栈容量超过默认大小而导致溢出。我在8G的内存下，用eclipse进行递归调用测试，递归11410次后抛SOF异常

栈溢出的原因：

- 递归调用
- 大量循环或死循环
- 全局变量是否过多
- 数组、List、map数据过大

------

**Android的OOM（Out Of Memory）**

> 当内存占有量超过了虚拟机的分配的最大值时就会产生内存溢出（VM里面分配不出更多的page）

一般出现情况：

- 加载的图片太多或图片过大时
- 分配特大的数组
- 内存相应资源过多没有来不及释放。

解决方法：

1. 在内存引用上做处理
2. 对图片进行边界压缩, 配合软引用使用
3. 显示的调用GC来回收内存例如`if(bitmapObject.isRecycled()==false) //如果没有回收 bitmapObject.recycle();`
4. 优化Dalvik虚拟机的堆内存分配
   - 增强堆内存的处理效率`VMRuntime.getRuntime().setTargetHeapUtilization(0.75);`
   - 设置堆内存的大小: `VMRuntime.getRuntime().setMinimumHeapSize(6 * 1024 * 1024);` 设置最小heap内存为6MB大小
5. 用LruCache和AsyncTask解决

### Math类

```
double floor(double a);     // 向下取整

double random();            // 产生随机数取值范围[0,1)

double ceil();              // 向上取整

long round(double a);       // 四舍五入
```

### String

**equals与==的区别**

- `==`：对于基本类型，比较的是它们的值。对于复合类型（直接在堆中分配空间），比较的是它们在内存中的地址。
- `equals`：该方法属于Object，而所有类都继承于Object这个基类，因此每个类都有这个方法。Object类中equals的默认实现是return (this == obj);，即默认是比较对象的内存地址。但在库中的一些类会覆盖重写equals这个方法，如：`String`、`Integer`、`Date`这些类中equals有自身的实现，而不再是比较类在堆内存中的地址。String中的equals，首先判断==，如果地址相同，那一定是返回true；如果地址不相同，再比较字符串字面值是否相等。

------

**Switch能否用string做参数?**

在Java7之前，`switch`只支持`byte`、`short`、`char`、`int`及其对应的封装类，以及`Enum`类型，在Java 7中，String类型被加上。

------

**String、StringBuffer与StringBuilder的区别**

1. 字符串是否可变
   - String: 使用字符数组保存字符串：private final char value[];关键字final决定了String对象不可变。
   - StringBuilder和StringBuffer继承自AbStractStringBuilder类，AbstractStringBuilder类也是使用字符数组保存字符串：char[] value;没有final，可知这两个对象都是可变的。
2. 线程安全
   - String对象不可变，也就可以理解为常量，显然线程安全。
   - StringBuffer对方法加了同步锁，因此是线程安全的。
   - StringBuilder没有加锁，是非线程安全的。

------

**String 常用的函数**

**split**

split函数原型为：`String[] split(String regex)`。参数`regex`不是一个简单的字符串，而是一个正则表达式。因此，对于正则表达式中的关键字你需要使用转意符`\`，例如.和`|`都是转义字符,必须得加`"\\"`：

```
如果用.作为分隔的话,必须写为String.split("\\.")，而不能直接这样String.split(".");
如果用|作为分隔的话,必须写为String.split("\\|")，而不能直接这样String.split("|");
如果在一个字符串中有多个分隔符,可以用|作为连字符，比如待分割的字符串为String s="my; name,is HuaChao"，如果希望把单词提取出来（以标点符号和空格为分割字符），可以写为：s.split(",| |;");注意， 两个|之间有空格，",| |;"表示，以,或空格以及;分割字符串。
```

**replace、replaceAll、replaceFirst**

- replace:原型为String replace(char oldChar, char newChar) ，即将所有的oldChar字符替换为newChar字符
- replace:原型为String replace(CharSequence target, CharSequence replacement) ，即将所有的target字符串替换为replacement字符串
- replaceAll：原型为String replaceAll(String regex, String replacement)，参数regex从名称可以看出，它是一个正则表达式。replacement为替换的新字符串，即将原字符串中，所有满足正则表达式regex的部分替换为replacement
- replaceFirst：原型为String replaceFirst(String regex, String replacement) ，跟replaceAll很像，只不过replaceFirst是替换第一个满足正则表达式regex的部分。

如下:

```
public static void main(String[] args) throws Exception { 
    String s="my.name.is.HuaChao";  
    System.out.println(s.replace('.', '*'));  
    System.out.println(s.replace(".", "*"));  
    System.out.println(s.replaceAll(".", "*"));  
    System.out.println(s.replaceFirst(".", "*"));  
 
}

// 结果
my*name*is*HuaChao 
my*name*is*HuaChao 
****************** 
*y.name.is.HuaChao
```

运行结果中，很好理解，第1个replace里面的参数是字符，不是正则表达式，replace会把所有的.字符替换为`*`；同样，第二个replace里面的参数是字符串，不是正则表达式；而replaceAll中，第一个参数是正则表达式，第二个参数是字符串，而正则表达式中的`.`是表示任意字符，因此，会把所有的字符替换为`*`；最后replaceFirst，只替换第一个字符。

------

**正则表达式**

如果我们需要从字符串中匹配出满足我们自定义的正则表达式的部分，就可以通过使用Pattern这个类。我们先看一个实际应用，假设我们要取出一个字符串中所有的字母，并显式出来：

```
public static void main(String[] args) { 
    String dataStr = "--->我是干扰字符<---M12v,L23f,d34"; 
    Pattern pattern = Pattern.compile("[a-zA-Z]"); 
    Matcher matcher = pattern.matcher(dataStr); 
    // 遍历匹配正则表达式的字符串 
    while (matcher.find()) { 
        // s为匹配的字符串 
        String s = matcher.group(); 
        System.out.println(s);  
    } 
}

// 结果
//  M v L f d
```

### List

**ArrayList**

ArrayList类的定义为:

```
public class ArrayList<E> extends AbstractList<E> implements List<E>,RandomAccess,Cloneable,Serializable
```

特性:

- 可变大小的数组
- 非线程安全
- 当更多的元素加入到ArrayList时，其大小会动态的增长。每次增长的空间是其size的50%。初始容量是10.
- 允许null元素

------

**LinkedList**

LinkedList类定义:

```
public class LinkedList<E> extends AbstractSequentialList<E> implements List<E>, Deque<E>, Cloneable, Serializable
```

LinkedList有如下特性：

- 是一个双链表
- 非线程安全
- 在添加和删除元素元素时具有比ArrayList更好的性能
- LinkedList还实现了Queue接口（非直接实现，是通过实现Queue的子接口Deque间接实现Queue），该接口比List提供了更多方法。包括从尾部添加元素：`offer(E)`、返回第一个元素但不出队:`peek()`、返回第一个元素并出队：`poll()`等。
- 允许null元素

由于LinkedList不同步,可以通过如下方式转化为同步的List

```
List list= Collections.synchronizedList(new LinkedList());
```

------

**Vector**

Vector类定义:

```
public class Vector<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, Serializable
```

Vector类有如下特性：

- Vector和ArrayList类似，但属于强同步类。
- 比ArrayList多了线程安全。
- 默认每次动态增加空间是当前大小的2倍；如果在构造函数Vector(int initialCapacity, int capacityIncrement)中指定了capacityIncrement，每次动态增加的大小为capacityIncrement
- 初始容量是10.
- 允许null元素

### Map

首先各个子类的继承关系

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/map.png)

类的定义:

```
//HashMap 
public class HashMap<K,V> 
    extends AbstractMap<K,V> 
    implements Map<K,V>, Cloneable, Serializable{} 
 
//Hashtable 
public class Hashtable<K,V> 
    extends Dictionary<K,V> 
    implements Map<K,V>, Cloneable, java.io.Serializable {} 
 
//ConcurrentHashMap 
public class ConcurrentHashMap<K, V> extends AbstractMap<K, V> 
        implements ConcurrentMap<K, V>, Serializable {} 
 
//TreeMap 
public class TreeMap<K,V> 
    extends AbstractMap<K,V> 
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable{} 
 
//LinkedHashMap 
public class LinkedHashMap<K,V> 
    extends HashMap<K,V> 
    implements Map<K,V>{}
```

------

**HashMap内部实现**

`HashMap`本质是数组加链表。根据`key`取得`hash`值，然后计算出数组下标，如果多个`key`对应到同一个下标，就用链表串起来。新插入的在前面。不保证映射顺序，特别是它不保证该顺序恒久不变。里面存放的是`Map.Entry`类，该类本质是个键值对。

- `HashMap`数据结构：根据`key`的`hashCode`来计算`hash`值，只要`hashCode`相同，计算出来的`hash`值就一样。出现`hash`冲突，就采用链表的方式，将相同`hash`值的对象用链表连接。
- `HashMap`存取：`put`新元素时，首先根据`key`的`hashCode`重新计算`hash`值（二次hash），根据这个新的`hash`值得到这个元素在数组的位置（下标），如果数组已经存放其他元素，那么该位置元素以链表形式存放，新加入的放链头，最先加入的在链尾。根据Key的hashCode二次hash的算法函数hash（int h），此方法加入了高位计算，防止低位不变而高位变化时造成的`hash`冲突。函数的具体实现如下(`>>>`表示右移1位并忽略符号位，空位以0补齐。而`>>`表示右移不忽略符号位，即相当于除以2)：

```
static int hash(int h){ 
    h ^= (h>>>20)^(h>>>12); 
    return h^(h>>>7)^(h>>>4); 
}
```

此时得到了二次hash，二次hash的主要目的就是将高位引入计算，使得计算出来的位置值与高位也有关。将二次hash值对数组长度取模运算这样一来元素的分布就比较均匀。但是，模运算的消耗比较大。在`HashMap`中这样做：调用`indexFor(int h,int length)`方法计算该对象应该保存在`table`数组的那个索引处：

```
static int indexFor(int h,int length){ 
    return h & (length-1); 
}
```

这个方法非常巧妙，它通过`h&(table.length-1)`来得到该对象的保存位，而`HashMap`底层数组的长度总是`2的n次方`（即只有一位上是1，其他位上是0），这是`HashMap`在速度上的优化。

`HashMap`扩容（`resize`、`rehash`）：由上面可知，每次数组扩容为原来的两倍。扩容会带来一个性能上的问题，就是每次扩容需要重新计算每个元素的位置。那么`HashMap`什么时候进行扩容呢？这个跟`loadFactor`（加载因子）有关，默认情况下`loadFactor`为`0.75`.即当`HashMap`元素超过`length*0.75`时，需要**扩大一倍**，然后重新计算元素在数组的位置，这是一个非常消耗性能的操作，所以，如果已经预知`HashMap`中元素个数那么预设元素个数能够有效提高`HashMap`性能。

- `Fail-Fast`(快速失败)机制： `HashMap`不是线程安全的，因此在使用迭代器过程中，其他线程修改了Map，那么将抛出`ConcurrentModificationException`异常，这就是`fail-fast`策略。实现原理为，通过`modCount`域，`modCount`顾名思义就是修改次数，对`HashMap`内容的修改都将增加这个值，在迭代器初始化过程会将这个值赋给迭代器的`expectedModCount`，迭代过程中，判断`modCount`跟`expectedModCount`是否相等，如果不相等就表示已经有其他的线程修改了Map。

------

**ConcurrentHashMap**

在HashMap的基础上,`ConcurrentHashMap`将数据分为多个`segment`，默认`16`个，然后每次操作对一个`segment`加锁，避免多线程锁的几率，提高并发效率。

------

**HashTable和HashMap**

- `HashMap`父类为`AbstractMap`，方法不同步，K，V可为null，添加新的kv，若k相同，则将新的v覆盖。
- `HashTable`父类为`Dictionary`，方法同步，k，v不可为null，添加新的kv，若k相同，则将新的v覆盖。

------

**TreeMap、HashMap、LinkedHashMap的区别**

- `TreeMap` 实现`SortMap`接口，能够把它保存的记录根据键排序，默认是**按键值升序排序**，也可以指定排序的比较器（通过构造器传入`Comparator`对象），当用`Iterator`遍历`TreeMap`时，得到的记录是排过序的。
- `LinkedhashMap`，是`HashMap`子类，保存了记录的插入顺序，在用`iterator`遍历`LinkedHashMap`时，先得到的记录肯定是先插入的，如果需要输出的顺序和输入的相同，那么`LinkedHashMap`可以实现。LRU算法里面使用到`LinkedHashMap`,之所以用这个类而不用`LinkedList`，主要是`LinkedHashMap`取值速度快，免去了`LinkedList`遍历搜索过程。

### Collections和Arrays

由于`collections`主要针对`Collection`对象, 先看看`Collection`子类继承结构

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/collection.png)

------

**Collections**

`Java.util.Collections`是一个包装类（工具类、帮助类），主要是针对集合类操作。它包含各种有关集合操作的静态多态方法。此类不能实例化，就像一个工具类，有如下功能：

1. 二分搜索算法进行查找
2. 为Collection添加不定数量参数作为子元素
3. 将一个List所有元素复制到另一个
4. 判断两个元素是否有相同的元素:
5. 获取Collection最大、最小元素
6. 用于对集合中的元素进行排序
7. 将线程不安全的Map、Set转为线程安全的对象
8. 返回单例

部分函数原型

```
static boolean addAll(Collection<? super T> c, T... elements); 
static int binarySearch(List<? extends T> list, T key, Comparator<? super T> c); 
static void copy(List<? super T> dest, List<? extends T> src); 
//如果没有相同的元素返回true 
static boolean disjoint(Collection<?> c1, Collection<?> c2); 
//最大最小值 
static T max(Collection<? extends T> coll, Comparator<? super T> comp); 
static T min(Collection<? extends T> coll, Comparator<? super T> comp); 
//逆序 
static Comparator<T> reverseOrder(Comparator<T> cmp) 
//返回只包含指定对象的单例Set 
static Set<T> singleton(T o); 
//返回只包含指定对象的单例列表 
static List<T> singletonList(T o); 
//返回只包含指定对象的单例Map 
static Map<K,V> singletonMap(K key, V value); 
//排序 
static void sort(List<T> list, Comparator<? super T> c); 
//交换指定位置的两个元素 
static void swap(List<?> list, int i, int j) 
//将线程不安全的Collection转为线程安全的Collection 
static Collection<T> synchronizedCollection(Collection<T> c); 
static List<T> synchronizedList(List<T> list); 
static Map<K,V> synchronizedMap(Map<K,V> m); 
static Set<T> synchronizedSet(Set<T> s);
```

------

**Arrays**

这些方法都是静态方法。主要是针对数组操作。Arrays跟Collections很像，包含如下功能：

- 二分搜索算法进行查找
- 将数组转List对象
- 复制数组指定范围的元素为一个新的数组
- 给数组指定范围的每个元素赋一个值，排序等等

函数部分原型:

```
//针对基本类型的查找 
//如果查找的数组类型是int[],则函数如下 
//其他的基本类型对应的二分搜索函数原型为把int替换指定的类型就好 
static int binarySearch(int[] a, int key); 
//针对基本类型，在指定范围进行二分搜索 
//其他基本类型类似 
static int binarySearch(int[] a, int fromIndex, int toIndex, int key); 
 
//针对类对象数组的二分搜索 
static int binarySearch(T[] a, T key, Comparator<? super T> c) 
 
//针对一个数组，复制其元素到一个新的数组， 
//并将新的数组返回. 
//其他基本类型相似，将float替换掉即可 
static float[] copyOf(float[] original, int newLength); 
 
//复制数组指定范围的元素到一个新的数组， 
//并将新的数组返回 
//其他基本类型类似，将byte替换即可 
static byte[] copyOfRange(byte[] original, int from, int to); 
//复制指定范围的类对象数组 
static  T[] copyOfRange(T[] original, int from, int to); 
 
//判断两个基本类型数组里面的元素是否相等 
//其他基本类型只需将char替换 
static boolean equals(char[] a, char[] a2); 
 
//判断两个类对象数组里面的元素是否相同 
static boolean equals(Object[] a, Object[] a2); 
 
//为基本类型数组里面的每一个元素赋相同的值 
//其他基本类型将boolean替换 
static void fill(boolean[] a, boolean val); 
//类数组一样 
static void fill(Object[] a, Object val) 
 
//返回hash码,基本类型替换long 
static int hashCode(long[] a); 
static int hashCode(Object[] a); 
 
//排序,基本类型替换byte 
static void sort(byte[] a); 
static void sort(byte[] a, int fromIndex, int toIndex); 
//模板类型排序 
static void sort(T[] a, Comparator<? super T> c); 
static void sort(T[] a, int fromIndex, int toIndex, Comparator<? super T> c); 
 
//toString，基本类型替换long 
static String toString(long[] a); 
static String toString(Object[] a);
```

### HashCode

**hashCode()方法**

因为`Object`类提供了`hashCode()`方法，因此，每个类对象都拥有`hashCode()`方法。而Object的 `hashCode`是一个`native`方法,我们就不去深究其具体的实现了。
`hashCode`方法主要作用是为了配合散列的集合一起工作。散列集合包括`HashSet`、`HashMap`以及`HashTable`。

------

**HashSet判断对象是否存在集合中**

我们知道，集合中是不允许重复元素存在的，当`HashSet`需要添加新的对象`obj`时，如何判断`obj`是否已经存在于集合中呢？

- 调用`obj.hashCode()`，得到对应的`hashcode`值。
- 如果集合中没有存储这个`hashcode`对应的对象，则直接添加。
- 如果集合中已经存储了这个`hashcode`对应的对象，则调用equals判断是否对象相同。

从上面过程可知，如果你重写`equals`方法，必须重写`hashCode`函数。因为：

如果只重写`equals`，根据你的规则将两个对象`equals`返回`true`，但是`hashCode`默认却不同，集合还是会添加新元素。

------

**HashSet存取**

`HashSet`是基于`HashMap`来实现的。`HashSet`相当于只利用`HashMap`的`Key`，而`value`使用一个 `static final`的`Object`对象标识。一次`HashSet`的存取相当于`HashMap`的一次存取，只不过`HashSet`只看重`Key`部分，不需要`Value`部分。因此，我们只需看接下来小节中的`HashMap`的put和get方法。

------

**HashMap的put和get方法**

我们知道，`HashMap`里面的结构是`数组+链表`。链表里面存储的元素就是键值对`HashMap.Entry<K,V>`对象。在存放`Key-Value`时，过程如下：

1. 首先根据`key`的`hashCode`找到对应数组的位置
2. 然后遍历该位置的链表，查找`key`是否已经存在
3. 如果`key`已经存在，则直接更新`value`,并将旧的`value`作为函数返回
4. 如果`key`不存在，则通过头插法，将新的键值对放入当前链表的第一个位置

**注意，null key总是放入数组的第0个位置，因为null的哈希码为0**

put方法已经讲解完，get方法相对就比较简单了:

1. 首先根据key的hashCode找到对应数组的位置
2. 然后遍历该位置的链表，查找key是否已经存在

### return和finally执行顺序

**结论**

1. 不管有木有出现异常，finally块中代码都会执行；
2. 当try和catch中有return时，finally仍然会执行；
3. 如果语句上的执行顺序是先return后finally，会先执行return后面的语句，这个语句的结果是最终的返回值result。result会被保存下来，再执行finally，待finally执行完成后，再结束函数，将result的值返回。这种情形的finally对变量的值修改不会影响最终的函数返回。
4. finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值。

### Override与Overload区别

**Override(重写、覆盖)**

`Override`是子类对父类的允许访问的方法的实现过程进行重新编写，`Override`一个函数需要注意以下几点：

- 返回值、函数名、形参都不能改变。即外壳不变，重写内在实现。
- 子类方法不能缩小父类方法的访问权限（反过来是可以的）
- `final`的方法不能被重写
- 声明为`static`的方法不能被重写，但是能够被再次声明
- 子类和父类在同一个包中，子类可以重写父类所有方法，除了声明为`private`和`final`的方法。
- 子类和父类不在同一个包中，子类只能够重写父类的声明为`public`和`protected`的非`final`方法。
- 重写的方法能够抛出任何非强制异常，无论被重写的方法是否抛出异常。
- 重写的方法不能抛出新的强制性异常，或者比被重写方法声明的更广泛的强制性异常，反之则可以。
- 构造方法不能被重写
- 如果不能继承一个方法，则不能重写这个方法( 父类的private方法)。

解释一下强制性异常和非强制性异常：

- 除了`RuntimeException`外，都是强制性异常
- 所谓强制性异常就是在编写程序的过程中必需在抛出异常的部分`try catch` 或者向上`throws`异常
- 所谓非强制性异常就和上面相反了。不过你当然也可以`try catch`或者`thows`，只不过这不是强制性的。

------

**Overload(重载)**

重载是在同一个类里面，**方法名字相同，而参数不同，返回类型可以相同也可以不同的多个方法**。每个重载的方法都必须有一个独一无二的参数类型列表。

重载规则如下：

- 被重载的方法必须改变参数列表；
- 被重载的方法可以改变返回类型；
- 被重载的方法可以改变访问修饰符；
- 被重载的方法可以声明新的或更广的检查异常；
- 方法能够在同一个类中或者在一个子类中被重载。

## Java 虚拟机

### CAS

> 所谓CAS(Compare And Swap) 即比较并交换

在`Intel`处理器中，比较并交换通过指令的 `cmpxchg` 系列实现。CAS有三个操作数：

- 内存位置（V）
- 预期原值（A）
- 新值(B)

如果内存位置`V`的值与预期`A`原值相匹配，那么处理器会自动将该位置值更新为新值`B`。否则，处理器不做任何操作。

无论哪种情况，它都会在 `CAS` 指令之前返回该位置的值。（在CAS的一些特殊情况下将仅返回CAS是否成功，而不提取当前值。）CAS有效地说明了“我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。

**CAS应用**

比较典型的应用就是`AtomicInteger`,可以看到，对`i++`和`i--`，都是通过`CAS`，并且通过一个死循环，`compareAndSet`函数内部就是通过`jni`操作`CAS`指令。直到`CAS`操作成功跳出循环。

```
private volatile int value; 
    /** 
     * Gets the current value. 
     * 
     * @return the current value 
     */ 
    public final int get() { 
        return value; 
    } 
    /** 
     * Atomically increments by one the current value. 
     * 
     * @return the previous value 
     */ 
    public final int getAndIncrement() { 
        for (;;) { 
            int current = get(); 
            int next = current + 1; 
            if (compareAndSet(current, next)) 
                return current; 
        } 
    } 
 
    /** 
     * Atomically decrements by one the current value. 
     * 
     * @return the previous value 
     */ 
    public final int getAndDecrement() { 
        for (;;) { 
            int current = get(); 
            int next = current - 1; 
            if (compareAndSet(current, next)) 
                return current; 
        } 
    }
```

### GC收集器

- Serial收集器
- ParNew收集器
- Parallel Scavenge收集器
- Serial Old收集器
- Parallel Old收集器
- CMS收集器
- G1收集器

------

**Serial收集器**

从名字可以看出，这个收集器是一个单线程的收集器。但是，它的“单线程”的意义并不仅仅说明它只会使用一个`CPU`或一条收集线程去完成垃圾收集工作，更重要的是，**在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束**。

`"Stop The World"`是在用户不可见的情况下，把用户正常工作的线程全部停掉，这对很多应用是难以接受的，试想一下，要是你的计算机每运行1小时就暂停响应5分钟，你会是什么样的心情！

运行示意图:

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/GC_Serial.png)

对于`"Stop The World"`给用户带来的不良体验，虚拟机设计者表示完全理解，但也表示非常委屈：“你妈妈在给你打扫房间时候，肯定也会让你老老实实地在椅子上或房间外呆着，如果她一边打扫，你一边乱扔纸屑，这房间还能打扫完？”

从`JDK1.3`开始，`HotSpot`虚拟机开发团队为消除或减少工作线程因内存回收而导致停顿的努力一直进行着。从`Serial`收集器到`Parallel`收集器，再到`Concurrent Mark Sweep（CMS）`乃至GC收集器的最前沿成功`Garbage First（G1）`收集器，用户线程停顿时间不短缩短，但是仍然无法完全消除！

**应用场景**

虽然`Serial收集器`看起来“老而无用、食之无味弃之可惜”，但实际上到目前为止，它依然是虚拟机运行在`Client模式下`的默认新生代收集器。它有着优于其他收集器的地方：简单高效（与其他收集器的单线程比）。

对于限定单个CPU的环境来说，`Serial收集器`由于没有线程交互的开销，专心做垃圾收集自然可以获得更高的单线程收集效率。

在用户的桌面应用场景中，分配给虚拟机管理的内存一般来说不会很大，收集几十兆甚至一两百兆新生代（仅仅是新生代使用的内存，桌面应用基本上不会再大了），停顿时间完全可以控制在几十毫秒最多一百毫秒以内，只要不是频繁发生，这点停顿还是可以接受的，所以Serial收集器对应运行`Client模式`下的虚拟机来说是一个很好的选择。

------

**ParNew 收集器**

**运行过程:**

`ParNew收集器`其实就是`Serial收集器`的**多线程版本**，除了使用多条线程进行垃圾收集之外，其余行为包括Serial收集器可用的所有控制参数（例如：-XX:SruvivorRatio、-XX:PretenureSizeThreshold、-XX:HandlePromotionFailure等）、收集算法、Stop The World、对象分配规则、回收策略等都与Serial收集器完全一样，在现实上，这两种收集器也共用了相当多代码。

`ParNew收集器`工作示意图:

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/GC_ParNew.png)

**应用场景**

`ParNew收集器`除了多线程收集之外，其他与Serial收集器相比并没有太多创新之处，但它却是许多运行在`Server模式下`的虚拟机中首选的新生代收集器。其中一个与性能无关但很重要的原因是，除了Serial收集器外，目前只有它能`与CMS收集器`配合工作。

`ParNew收集器`在单CPU的环境中绝对不会有比`Serial收集器`更好的效果。甚至由于存在线程交互的开销，该收集器在通过超线程技术实现两个CPU环境中都不能百分百地保证可以超越`Serial收集器`。当然，随着CPU数量增加，它对于GC时，系统资源的有效利用还是很有好处。它默认开启的收集线程数与CPU数量相同，在CPU非常多（例如32个，现在CPU动辄就4核加超线程，服务器超过32个逻辑CPU的情况越来越多了）环境下，可以使用`-XX:ParalleGCThreads`参数来限制垃圾收集的线程数。

------

**Parallel Scavenge收集器**

`Parallel Scavenge收集器`是一个新生代收集器，它也是使用**复制算法**的收集器，又是并行的多线程收集器….看上去和ParNew都一样，那它有啥特别的地方呢？

`Parallel Scavenge收集器`的特点是它的关注点与其他收集器不同，`CMS`等收集器的关注点是**尽可能第缩短垃圾收集时用户线程停顿时间**，而`Parallel Scavenge收集器`的目标则是**达到一个可控制的吞吐量**。所谓的吞吐量就是：

> CPU用于运行用户代码的时间与CPU总消耗时间的比值，即：吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间)

虚拟机总共运行`100分钟`，其中垃圾收集消耗掉`1分钟`，那吞吐量就是`99%`。

------

**Serial Old收集器**

`Serial Old`是`Serial收集器`的老年代版本，它同样是一个单线程收集器，使用“标记-整理”算法，这个收集器的主要意义也是在于给`Client模式下的虚拟机`使用。如果在Server模式下，它主要还有两大用途：

- 在JDK1.5以及之前版本中与`Parallel Scavenge收集器`搭配使用
- 作为CMS收集器的后备预案，在并发收集发生`Concurrent Mode Failure`时使用

运行示意图和`Serial收集器类似`

------

**Parallel Old收集器**

`Parallel Old收集器`是`Parallel Scavenge收集器`的老年代版本，使用多线程和“标记-整理”算法。这个收集器在JDK1.6中才开始提供。

Parallel Old收集器运行示意图如下：

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/ParallelOld.png)

------

**CMS收集器**

`CMS（Concurrent Mark Sweep）收集器`是一种以获取最短回收停顿时间为目的的收集器。目前很大一部分的Java应用集中在互联网站或者B/S系统服务器上，这类应用尤其重视服务的响应速度，希望系统停顿时间最短，以给用户最好的体验。`CMS收集器`就非常符合这类应用的需求。

从名字（包含“Mark Sweep”）上就可以看出，`CMS收集器`是基于“标记-清除”算法实现的，它的运作过程相对前面几种收集器来说更复杂一些，整个过程分为4个步骤：

- 初始标记
- 并发标记
- 重新标记
- 并发清除

其中，初始标记、重新标记着两个步骤仍然需要`“Stop The World”`。初始标记仅仅只是标记一下`GC Roots`能直接关联到的对象，速度很快。并发标记阶段就是进行`GC Roots Tracing`的过程。而重新标记阶段则是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间会比初始标记稍长一些，但远比并发标记时间短。

由于整个过程中耗时最长的**并发标记和并发清除过程**收集器线程都可以与用户线程一起工作，所有总体上来说，CMS收集器的内存回收过程是与用户线程一起并发执行的。CMS运作步骤如下：

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/CMS.png)

CMS是一款优秀的收集器，它的主要优点从名字上体现出来：并发收集、低停顿。但是CMS还远达不到完美程度，它有以下3个明显的缺点：

- `CMS收集器`对`CPU`资源非常敏感。在并发阶段，它虽然不会导致用户线程停顿，但是会因为占用一部分线程（或者说CPU资源）而导致应用程序变慢，总吞吐量会降低。
- `CMS收集器`无法处理浮动垃圾，可能出现`Concurrent Mode Failure`失败而导致另一次`Full GC`产生。由于CMS并发清理阶段用户线程还在运行着，伴随程序运行自然就还会有新的垃圾不断产生，这一部分垃圾出现在标记过程之后，CMS无法再当次收集中处理掉它们，只好留待下一次GC时再清理掉。这部分垃圾就称为“浮动垃圾”。因此，CMS不能像其他收集器那样等到老年代几乎完全被填满再进行收集，CMS需要预留一部分空间。
- 由于CMS基于`“标记-清除”`算法，意味着收集结束时会有大量空间碎片产生。

------

**G1收集器**

`G1（Garbage First）收集器`是当今收集器技术发展的最前沿成果之一。G1是面向服务端应用的垃圾收集器，与其他GC收集器相比，G1具备如下特点：

- 并行与并发：充分利用多CPU、多核环境下的硬件优势，使用多个CPU（CPU或CPU核心）来缩短Stop-The-World停顿时间。部分其他收集器需要停顿Java线程执行的GC动作，G1仍然能通过并发方式让Java程序继续执行。
- 分代收集：与其他收集器一样，分代概念在G1中依然得以保存。
- 空间整合：与CMS的“标记-清理”算法不同，G1从整体上看是基于“标记-整理”算法实现的收集器，从局部上看是基于“复制”算法实现的，这两种算法意味着G1运作期间不会产生内存空间碎片。
- 可预测的停顿：这是G1相对于CMS的另一大优势，降低停顿时间是G1和CMS共同关注点，但G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得从超出N毫秒，这几乎已经是实时Java的垃圾收集器的特征了。

G1运作大致可划分为以下几个步骤：

- 初始标记
- 并发标记
- 最终标记
- 筛选回收

运行示意图如下：

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/GC_G1.png)

### 内存模型和分区

**逻辑内存模型**

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/memorymodel.png)

------

**内存分区**

- `程序计数器`: 较小的内存空间。线程私有。可以看成是**当前线程所执行的字节码的行号指示器**。通过改变这个计数器的值来选取下一条需要执行的字节码指令。分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖计数器。此内存区域**是唯一一个在JVM规范中没有规定任何OutofMemoryError情况的区域**。
- `Java虚拟机栈`: 线程私有，生命周期与线程相同。描述的是Java方法执行的内存模型：每个方法执行时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每个方法从调用到执行完成，就对应一个栈帧在虚拟机栈中的**入栈**和到**出栈**的过程。会抛出`StackOverflowError`和`OutOfMemoryError`.
- `本地方法栈`: 功能与虚拟机栈类似。区别在于本地方法栈为`native`方法服务。
- `Java堆`: `JVM`所管理的内存中最大的一块。所有线程所共享。可分为：`新生代`和`老年代`。新生代可再细分为：`Eden空间`、`From Survivor空间`、`To Survivor空间`。有`OutOfMemoryError`异常。
- `方法区`: 跟`Java`堆一样，是各个线程共享区域。存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。运行时常量池是方法区一部分。
- `直接内存`: 不属于虚拟机运行时数据区的一部分。`NIO`引入了一种**基于通道与缓冲区的IO方式**。他可以使用`Native`函数库直接分配堆外内存，然后通过一个存储在`Java`堆中的`DirectByteBuffer`对象作为这块内存的引用进行操作。避免`Java`堆和`Native`堆之间来回复制数据，在某种场景中显著提高性能。由于不在堆中分配，因此不受到堆大小限制。但既然是内存总有会被用完时候，因此会抛出`OutOfMemoryError`。

### 新生代老年代

**新生代于老年代占空间比例**

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/newold.png)

堆被划分为**新生代**和**老年代**。默认比例为`1:2`（可以通过`–XX:NewRatio` 设定）。新生代又分为`Eden`、`From Survivor`、`To Survivor`。这样划分的目的是为了使 ˚能够更好的管理堆内存中的对象，包括内存的分配以及回收。

新生代分为的三个部分, 默认比例为`Eden:from:to=8:1:1`（可以通过参数`–XX:SurvivorRatio` 来设定，–`XX:SurvivorRatio =8`表示`Eden`与一个`Survivor`空间比例为8:1）

------

**存活对象的拷贝**

一般新建的对象会分配到`Eden`区。这些对象经过第一次`Minor GC`后，如果仍然存活，将会被移到`Survivor`区。在`Survivor`每熬过一轮`Minor GC`年龄就增加1。

当年龄达到一定程度时(年龄阈值，默认为`15`，可以通过`-XX:MaxTenuringThreshold`来设置)，就会被移动到老年代。

`from`和`to`之间会经常互换角色，`from`变成`to`，`to`变成`from`。每次`GC`时，把`Eden`存活的对象和`From Survivor`中存活且没超过年龄阈值的对象复制到`To Survivor`中，`From Survivor`清空，变成`To Survivor`。

**Minor GC与Full GC**

`Java`中的堆也是`GC`收集垃圾的主要区域。GC分为两种：

- Minor GC
- Full GC

`Minor GC`是发生在**新生代中**的垃圾收集动作，所采用的是**复制算法**，因为`Minor GC`比较频繁，因此一般回收速度较快。`Full GC` 是发生在**老年代**的垃圾收集动作，所采用的是**标记-清除算法**，速度比Minor GC慢10倍以上。

大对象直接进入**老年代**。比如很长的字符串以及数组。通过设置`-XX：PretenureSizeThreshold`，令大于这个值得对象直接在老年代分配。这样做是为了避免在`Eden`和两个`Survivor`之间发生大量的内存复制。

> 什么时候发生 Minor GC？什么时候发生Full GC？
> 当新生代Eden区没有足够的空间进行分配时，虚拟机将发起一次Minor GC。
> 老年代空间不足时Full GC

### 判断对象是否存活

判断哪些对象是存活的，哪些对象消亡的，典型的有两种方法：

- 引用计数算法
- 可达性分析算法

------

**引用计数**

给对象添加一个引用计数器，每当有一个地方引用它时，计数器+1，引用失效计数器-1；**任何时候计数器为0**的对象就是不可能再被使用。这有个问题是，两个对象相互引用导致两个对象都无法被回收。

------

**可达性分析**

通过一系列的`GC Roots`对象作为起点，从这些节点开始向下搜索。搜索所走过的路称为引用链。当一个对象到`GC Roots`没有任何引用链相连时，则证明此对象不可用。

可作为`GC Root`的对象有：

- 虚拟机栈（栈帧的本地变量表）中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中JNI引用对象

即使在可达性分析中不可达的对象，也并非是“非死不可”，这时候他们暂时处于“缓刑”阶段。要真正宣告一个对象死亡，需要经历两个阶段：

1. 如果对象在进行可达性分析后发现没有与`GC Roots`相连接的引用链，那它会被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行`finalize()`方法。当对象没有覆盖`finalize()`方法，或者`finalize()`方法已经被虚拟机调用过，虚拟机将这两种情况都视为“没有必要执行”。
2. 如果这个对象被判断为有必要执行`finalize()`方法。那么这个对象会被放到一个`F-Queue`队列中，并在稍后由一个虚拟机自动建立的、优先级低的`Finalizer线程`去执行它，这里的“执行”是指虚拟机会触发这个方法，但并不承诺等待它运行结束。这是为了防止`finalize()`方法执行缓慢使得`F-Queue`队列其他对象永久等待。

因此，对象可以在`finalize()`方法里把自己赋值给一个变量，以达到“自救”的目的，但是这样的“自救”只能用一次（虚拟机只会调用一次`finalize()`方法）。

### GC的三种收集方法

**1.标记-清除**

分为“标记”和“清除”两个阶段，首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象，他的标记过程在上面几行中已经提到过。

不足：

- 标记和清除两个过程效率不高
- 标记清除后产生大量不连续的内存碎片。
- 该算法主要用在老年代区域。

------

**2.复制算法**

将内存分为两部分，每次使用其中一块，当这块内存用完，就将还存活的对象复制到另一块上面。

不足：

- 浪费一半内存

通常用在新生代区域中，有个改进的方法是将新生代分为Eden、From Survivor、To Survivor。

------

**3.标记-整理**

标记过程和标记清除一样。但后续步骤不是直接对可回收对象进行清理，而是让所有存活对象向一端移动。

主要用在老年代区域。

### 类加载过程

类加载的5个过程分为:

- 加载
- 验证
- 准备
- 解析
- 初始化

------

**1.加载**

1. 通过类的全限定名来获取定义此类的二进制字节流
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

**2.验证**

1. 文件格式验证
2. 元数据验证
3. 字节码验证
4. 符号引用验证。

**3.准备**

为类变量（`static`）分配内存并设置类变量的初始值。

> 注意，实例变量并不在这个阶段分配内存。为类变量设置初始值并不是定义的值。
> 比如static int value = 123;那么变量value在准备阶段过后初始值为0，而不是123。值123是在`<clinit>()`方法中赋予。

**4.解析**

将常量池内的符号引用转为直接的引用.

**5.初始化**

按照`static块`和`static变量`在文件中的出现顺序，合并到`<clinit>()`方法中。实例变量由`<init>()`函数赋值。

### 静态分派 动态分派

**静态分派**

**概念:**静态分派与重载有关，虚拟机在重载时是通过参数的静态类型，而不是运行时的实际类型作为判定依据的；静态类型在编译期是可知的；

看如下例子:

```
public class Test { 
    static abstract class Human { 
    } 
 
    static class Man extends Human { 
    } 
 
    static class Woman extends Human { 
    } 
 
    public void sayHello(Human guy) { 
        System.out.println("hello,guy!"); 
    } 
 
    public void sayHello(Man guy) { 
        System.out.println("hello gentleman!"); 
    } 
 
    public void sayHello(Woman guy) { 
        System.out.println("hello lady!"); 
    } 
 
    public static void main(String[] args) { 
        Human man = new Man(); 
        Human woman = new Woman(); 
 
        Test test = new Test(); 
        test.sayHello(man); 
        test.sayHello(woman); 
    } 
}
```

输出结果为 :

```
hello,guy! 
hello,guy!
```

稍微有Java开发经验的人都能得到正确的答案。但是为什么会选择执行参数类型为`Human`的重载呢？在解决这个问题之前，我们先按如下代码定义两个重要概念：

`Human man = new Man();`

上面代码中，`Human`称为**变量的静态类型**；后面的`Man`则称为**变量的实际类型**。
静态类型和实际类型在程序中都可以发生一些变化，区别是：

- 静态类型的变化仅仅在使用时发生，变量本身的静态类型不会被改变，并且最终的静态类型是编译期可知的
- 实际类型变化的结果在运行期才能确定，编译器在编译程序时并不知道一个对象的实际类型是什么。例如下面代码：

```
//实际类型变化 
Human man=new Man(); 
man=new Woman(); 
 
//静态类型变化 
test.sayHello((Man)man); 
test.sayHello((Woman) man)
```

使用哪个重载版本，完全取决于**传入参数的数量**和**数据类型**。代码中刻意定义两个静态类型相同，但实际类型不同的变量，但虚拟机在重载时，是通过参数的**静态类型**而不是**实际类型**作为判断依据。并且静态类型在编译期是可知的，因此，在编译阶段，`javac`编译器会根据参数的静态类型决定使用哪个重载版本，所以选择了`sayHello(Human)`作为调用目标，并把这个方法的符号引用写到`man()`方法里面两条`invokevirtual`指令参数中。

> 所有依赖静态类型来定位方法执行版本的分派动作称为静态分派。静态分派的典型应用就是方法重载。

**重载方法的匹配优先级**

**基本类型中**，以`char`为例，按照如下优先级：

`char>int>long>float>double>Character>Serializable>Object>...`其中…为变长参数，

注意：`char`到`byte`或`short`之间的转换是不安全的

**引用类型中**，需要根据继承关系进行匹配，注意只跟其编译时类型即静态类型相关。

------

**动态分派**

动态分派与`重写(Override)`相关。同样，看一个例子：

```
public class Test { 
    static abstract class Human { 
        protected abstract void sayHello(); 
    } 
 
    static class Man extends Human { 
        @Override 
        protected void sayHello() { 
            System.out.println("man say hello"); 
 
        } 
    } 
 
    static class Woman extends Human { 
        @Override 
        protected void sayHello() { 
            System.out.println("woman say hello"); 
 
        } 
    } 
 
    public static void main(String[] args) { 
        Human man = new Man(); 
        Human woman = new Woman(); 
 
        man.sayHello(); 
        woman.sayHello(); 
 
        man = new Woman(); 
        man.sayHello(); 
    } 
}
```

运行结果:

```
man say hello 
woman say hello 
woman say hello
```

代码很简单，基本都能回答正确。但是现在问题是，虚拟机是如何知道调用哪个方法？

显然这里不能再根据静态类型决定，因为静态类型同样都是`Human`的两个变量`man`和`woman`在调用`sayHello()`方法时执行了不同的行为，并且变量`man`在两次调用中执行了不同的方法。

导致这个现象的原因很明显，是这两个变量的**实际类型**不同，java虚拟机是如何根据实际类型来分派方法执行版本呢？

看一下这两句代码：

```
Human man=new Man(); 
Human woman=new Woman();
```

这两个对象是即将要执行sayHello()方法的所有者，称为接受者。

由于`invokevirtual`指令执行第一步就是在运行期间确定接受者的实际类型，所以两次调用中的`invokevirtual`指令**把常量池中类方法符号引用解析到不同的直接引用上**，这个过程就是Java语言中方法重写的本质。这种运行期**根据实际类型确定方法执行版本的分派过程称为动态分派**。

### 类与类加载器

类加载器虽然只用于实现类的加载动作，但它在Java程序中起到的作用却远远不限于类加载阶段。对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名空间。简单说：

- 比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义
- 否则，即使两个类来源于同一个Class文件，被同一个虚拟机加载，只要他们的类加载器不同，那这两个类就必定不等。

这里指的“相等”，包括代表`Class`对象的`equals()`方法、`isAssignableFrom()`方法、`isInstance()`方法的返回结果，而包括使用`instanceof`关键字做对象所属关系判定等情况。

如果没有注意到类加载器的影响，在某些情况下，可能会产生具有迷惑性的结果，看个例子：

```
public class Test { 
    static ClassLoader myLoader = new ClassLoader() { 
        @Override 
        public Class<?> loadClass(String name) throws ClassNotFoundException { 
 
            if (!name.equals("com.szysky.Test")) 
                return super.loadClass(name); 
 
            try { 
                String fileName = name.substring(name.lastIndexOf(".") + 1) 
                        + ".class"; 
 
                InputStream is = getClass().getResourceAsStream(fileName); 
                if (is == null) { 
                    return super.loadClass(fileName); 
                } 
                byte[] b = new byte[is.available()]; 
                is.read(b); 
                return defineClass(name, b, 0, b.length); 
 
            } catch (IOException e) { 
                throw new ClassNotFoundException(name); 
            } 
        } 
    }; 
 
    public static void main(String[] args) throws Exception { 
 
        Object obj = myLoader.loadClass("com.szysky.Test"); 
        System.out.println(obj); 
        System.out.println(obj instanceof com.szysky.Test); 
    } 
 
}
```

运行结果:

```
class com.szysky.Test 
false
```

从第二句发现，这个对象与类`com.szysky.Test`做所属类型检查时返回了false，这是因为虚拟机中存在了两个Test类，一个是由系统应用程序类加载器加载的，另一个是由我们自定义的类加载器加载，虽然都是来自同一个class文件，但依然是两个独立的类，对象所属类型检查结果自然为false。

------

**自定义类加载器**

首先，定义一个类加载器MyClassLoader.java

```
public class MyClassLoader extends ClassLoader { 
    // 类加载器的名称 
    private String name; 
    // 类存放的路径 
    private String classpath = "E:/"; 
 
    MyClassLoader(String name) { 
        this.name = name; 
    } 
 
    MyClassLoader(ClassLoader parent, String name) { 
        super(parent); 
        this.name = name; 
    } 
 
 
 
    @Override 
    public Class<?> findClass(String name) {  
        byte[] data = loadClassData(name); 
        return this.defineClass(name, data, 0, data.length); 
    } 
 
    public byte[] loadClassData(String name) { 
        try { 
            name = name.replace(".", "//"); 
            System.out.println(name); 
            FileInputStream is = new FileInputStream(new File(classpath + name 
                    + ".class")); 
            byte[] data = new byte[is.available()]; 
            is.read(data); 
            is.close(); 
            return data; 
 
        } catch (Exception e) { 
            e.printStackTrace(); 
        } 
        return null; 
    } 
}
```

定义待加载的类:

```
public class TestObject { 
    public void print() { 
        System.out.println("hello ClassLoader"); 
 
    } 
}
```

定义测试类

```
public class Test { 
 
    public static void main(String[] args) throws InstantiationException, 
            IllegalAccessException, ClassNotFoundException { 
        // 新建一个类加载器 
        MyClassLoader cl = new MyClassLoader("myClassLoader"); 
        // 加载类，得到Class对象 
        Class<?> clazz = cl.loadClass("com.szysky.TestObject"); 
        // 得到类的实例 
        TestObject test= (TestObject) clazz.newInstance(); 
        test.print(); 
    } 
 
}
```

可以正常输出语句.

### 双亲委派模型

从虚拟机的角度来讲，只存在**两种不同的类加载器**

- 启动类加载器（`Bootstrap ClassLoader`）。使用C++语言实现（针对HotSpot虚拟机而言），是虚拟机自身的一部分。
- 所有其他的类加载器。使用Java语言实现，独立于虚拟机外部，并且全部继承自抽象类`java.lang.ClassLoader`

从Java程序员角度来看，类加载器还可以划分的更细致一点，绝大部分Java程序都会使用到以下3种系统提供的类加载器：

- 启动类加载器
- 扩展类加载器
- 应用程序类加载器

------

**启动类加载器**

这个类加载器将存放在`<JAVA_HOME>\lib`目录中的，或者被`-Xbootclasspath`参数所指定路径中的，并且是虚拟机识别的（仅按文件名识别，如：`rt.jar`，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给引导类加载器，那直接使用null代替即可，如`java.lang.ClassLoader.getClassLoader()`方法所示：

```
@CallerSensitive 
public ClassLoader getClassLoader() { 
    ClassLoader cl = getClassLoader0(); 
    if (cl == null) 
        return null; 
    SecurityManager sm = System.getSecurityManager(); 
    if (sm != null) { 
        ClassLoader.checkClassLoaderPermission(cl, Reflection.getCallerClass()); 
    } 
    return cl; 
}
```

**扩展类加载器(Extension ClassLoader)**

这个类加载器由`sum.misc.Launcher$ExtClassLoader`实现，它负责加载`<JAVA_HOME>\lib\ext`目录中的，或者被`java.ext.dirs`系统变量所指定的路径中的所有类库，开发者可以直接使用扩展类加载器。

**应用程序类加载器（Application ClassLoader**

这个类加载器由`sum.misc.Launcher$AppClassLoader`实现。由于这个类加载器是`ClassLoader`中的`getSystemClassLoader()`方法的返回值。所以，一般也称它为系统类加载器。它负责加载用户类路径`（ClassPath）`上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

------

**双亲委派模型**

如下图所示，这种类加载器之间的层次关系，称为类加载器的双亲委派模型。

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/classLoad.png)

双亲委派模型要求：

> 除了顶层启动类加载器以外，其余的类加载器都应当有自己的父类加载器。

这里类加载器之间的父子关系一般不会以继承的关系来实现，而是都使用**组合关系来复用父加载器的代码**。它并不是一个强制性的约束模型，而是Java设计者推荐给开发者的一种类加载器实现方式。

**双亲委派模型工作过程**

如果一个类加载器收到了类加载请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父类加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去加载。

使用双亲委派模型来组织类加载器之间的关系，有一个显而易见的好处就是：

> Java类随着它的类加载器一起具备了一种带优先级的层次关系

例如：类`java.lang.Object`，它存放在`rt.jar`之中，无论哪一个类加载器要加载这个类，最终都是委派给处于模型顶端的启动类加载器进行加载，因此`Object`类在程序的各种类加载器环境中都是同一个类。相反，如果没有使用双亲委派模型，由各个类加载器自行去加载的话，如果用户自己编写了一个称为`java.lang.Object`的类，并放在`ClassPath`中，那系统中将出现多个不同的`Object`类，Java类型体系中最基础的行为也就无法保证，应用程序也将变得一片混乱。

实现双亲委派模型的代码都集中在`java.lang.ClassLoader的loadClass()`方法中，逻辑清晰易懂：

- 先检查是否已经被加载过，若没有加载，则调用父加载器的loadClass()方法
- 若如加载器为空，则默认使用启动类加载器作为父加载器
- 如果父加载失败，抛出ClassNotFoundException异常之后，再调用自己的findClass()方法进行加载

以下是`ClassLoader`的`loadClass()`方法：

```
@Override 
protected Class<?> loadClass(String name, boolean resolve) 
        throws ClassNotFoundException { 
    Class<?> c = findLoadedClass(name); 
    if (c == null) { 
        try { 
 
            if (parent != null) { 
                c = parent.loadClass(name, false); 
            }else{ 
                c=findBootstrapClassOrNull(name); 
            } 
        } catch (ClassNotFoundException e) { 
            //如果父类加载器抛出ClassNotFoundException， 
            //说明父类加载器无法完成加载请求 
        } 
        if(c==null){ 
            //在父类加载器无法加载的时候 
            //再调用本身的findClass方法进行类加载 
            c=findClass(name); 
        } 
    } 
    if(resolve){ 
        resolveClass(c); 
    } 
    return c; 
}
```

### 对象创建 内存分布 访问定位

对象在JVM中是如何创建、如何布局以及如何访问的。讨论这个问题需要限定在具体的虚拟机和集中在某一个内存区域上才有意义。我们这个所说的是**Sun的HotSpot虚拟机的Java堆内存区域**，深入探讨HotSpot虚拟机在Java堆中对象的分配、布局和访问全过程。

------

**对象的创建**

在语言层面上，创建对象（例如克隆、反序列化）通常仅仅是一个`new`关键字而已，而在虚拟机中，对象（文中探讨的对象限于普通对象，不包括数组和Class对象等）的创建又是怎样一个过程呢？

1. `new指令开始:` 虚拟机遇到一个new指令时，首先去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并检查这个符号引用代表的类是否已经被加载、解析和初始化过。如果没有，需要先执行相应的类加载过程。参考类加载的五个过程。

2. `为对象分配内存:` 在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定（下一节介绍如何完全确定）。**为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来**

   - `划分空间:`假设`Java`堆中内存是绝对规整的，所有用到的内存在一边，空闲的内存在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式成为`“指针碰撞”`。如果Java堆中的内存**并不是规整的**，已使用的内存和空闲的内存相互交错，那就**没有办法简单地进行指针碰撞了**，此时：虚拟机就必须维护一个列表，记录上那些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为`“空闲列表”`。选择哪种分配方式由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定。因此，在使用`Serial`、`ParNew`等带`Compact`过程的收集器时，系统采用的分配算法是指针碰撞，而使用`CMS`这种基于`Mark_sweep`算法的收集器时，通常采用空闲列表。

   - ```
     划分的线程安全:
     ```

     除如何划分可用空间外，还有另外一个需要考虑的问题是对象创建在虚拟机中是非常频繁的操作，即使是仅仅修改一个指针所指向的位置，在并发情况下是线程不安全的，可能出现正在给对象A分配内存，指针还没有来得及修改，对象B又同时使用了原来的指针来分配内存的情况。解决这个问题有两种方案：

     - 对分配内存空间的动作进行同步处理——实际上虚拟机采用`CAS`配上失败重试的方式保证更新操作的原子性；
     - 把内存分配动作按照线程划分在不同的内存空间之中进行，即每个线程在Java堆中预先分配一小块内存，称为本地线程分配缓冲（Thread Local Allocation Buffer ，TLAB）。哪个线程要分配内存，就在哪个线程的`TLAB`上分配，只有`TLAB`用完并分配新的`TLAB`时才需要同步锁。虚拟机是否使用`TLAB`，可以通过`-XX:+/-UseTLAB`参数来设定。

3. `内存空间初始化为零值:`内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），如果使用`TLAB`，这一工作过程也可以提前至`TLAB`分配时进行。

   - 这一步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型对应的零值。
   - 接下来，虚拟机要对对象进行必要的设置，例如：这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的GC分代年龄等信息。这些信息存放在对象的对象头之中。根据虚拟机当前的运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。上面工作都完成后，从虚拟机的角度来看，一个新的对象已经诞生了，但从Java程序来说，对象创建才刚刚开始，所有的字段都还为零，需要进行一些初始化操作。

小结: 对象的创建过程如下:

- 虚拟机首先需要进行类加载检查
- 检查通过之后，根据类加载完成后确定的内存大小，为对象分配内存
- 接着，需要对分配到的内存空间都初始化为零值
- 然后，虚拟机要对对象设置一些基本信息，如对象是那个类的实例、对象的哈希码、对象的GC分代年龄信息、如何才能找到类的元数据信息等，到这里虚拟机创建对象的工作已经完成
- 最后，从程序的角度，我们还需要对对象进行初始化操作。

------

**对象的内存布局**

在`HotSpot`虚拟机中，对象在内存中存储的局部可以分为3块区域：

- 对象头（Header）
- 实例数据（Instance Data）
- 对齐填充（Padding）

> `HotSpot`虚拟机的对象头包括两部分信息

- 用于存储对象自身的运行时数据
  - 哈希码（HashCode）
  - GC分代年龄
  - 锁状态标志
  - 线程持有的锁
  - 偏向线程ID
  - 偏向时间戳
    - 这部分数据长度在32位和64位的虚拟机中分别为32bit和64bit，官方称它为`“Mark Word”`。对象需要存储的运行时数据很多，其实已经超出了32位、64位Bitmap结构能够记录的限度。但是对象头信息是与对象自身定义的数据无关的额外存储成本，考虑到虚拟机的空间效率，`Mark Word`被设计成为一个固定的数据结构以便在极小的空间存储尽量多的信息，它会根据对象的状态复用自己的存储空间。
- 类型指针，即对象指向它的类元数据的指针
  - 对象头的另外一个部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例，并不是所有的虚拟机实现都必须在对象数据上保留类型指针（还有通过句柄的方式）。另外，如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通Java对象的元数据信息确定对象的大小，但是从数组的元数据中却无法确定数组的大小。

> 实例数据部分

接下来的实例数据部分是对象真正存储的有限信息，也是程序代码中所定义的各种类型字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。这部分的存储顺序会受到虚拟机分配参数（`FieldAllocationStyle`）和字段在Java源码中定义顺序的影响。

HotSpot虚拟机默认的分配策略为:

`longs/doubles、ints、shorts/chars、bytes/booleans、oop(Ordinary Object Pointers)`

从分配策略中可以看出，相同宽度的字段总是被分配到一起。在满足这个前提的条件下，在父类中定义的变量会出现在子类之前。如果`CompactFields`参数值为`true`，那么子类之中较窄的变量也可能会插入到父类变量的空隙之中。

> 第三部分对齐填充并不是必然存在的，

也没有特别的含义，它仅仅起着占位符的作用。由于`HotSpot VM`的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数，因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

------

**对象的访问定位**

建立对象是为了使用对象，我们的Java程序需要通过`栈上的reference`数据来`操作堆上的具体对象`。由于`reference`类型在Java虚拟机规范中只规定了一个指向对象的引用，并没有定义这个引用应该通过何种方式去定位、访问堆中的对象的具体位置，所以对象访问方式也是取决于虚拟机实现而定的。目前主流的访问方式有**使用句柄**和**直接指针**两种。如果使用句柄访问的话，那么Java堆中将会划分出一块内存来作为**句柄池**，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据与类型数据各自的具体地址信息。

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/handlepoll.png)

如果是直接指针访问，那么Java堆对象的布局中就必须考虑如何放置访问类型数据的相关信息，而reference中存储的直接就是对象地址。

![img](http://szysky.com/2016/11/16/%E9%9D%A2%E7%BB%8F%E4%B9%8BJava%E7%AF%87/pointpoll.png)

这两种对象访问方式各有优势：

- 使用句柄来访问的最大好处就是`reference`中存储的是**稳定的句柄地址**，在对象被移动（垃圾收集时移动对象是非常普遍的行为）是只会改变句柄中的实例数据指针，而reference本身不需要修改。
- 使用直接指针访问方式的最大好处就是速度更快，它节省了一次指针定位的时间开销，由于对象的访问在Java中非常频繁。`Sun HotSpot`虚拟机采用的是第二种方式。