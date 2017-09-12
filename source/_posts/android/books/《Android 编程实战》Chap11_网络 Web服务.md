title: 《Android 编程实战》Chap11_网络 Web服务
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---
> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## Android上的网络调用

虽然`Android`同时支持`TCP`和`UDP`通信, 但应用程序的大部分网络调用都是建立在`TCP`之上的`HTTP`请求完成的.

网络操作的两个比较重要的规则:

1. 永远不要在主线程做耗时操作
2. 在`Service`而不是`Activity`中执行网络操作. 因为有很多情况下, 在`Activity`中执行网络操作, 很多时候需要考虑`Activity`快速切换的状态. 比如用户按了主屏幕键, 然后1秒后又回到了应用程序.

### HttpUrlConnection

`Android`提供了两个用于`HTTP`通信的API. `Apache`的`HttpClient`和`HttpUrlConnection`. 两者都能提供相同的功能. 但是推荐`HttpUrlConnection`, 因为谷歌一直在对其维护. 比如透明的响应压缩, 响应缓存.

`Android 4.0(ICS)`提供响应缓存功能, 所以如果要支持早期的版本, 开发者需要使用手动实现通过反射来初始化缓存. 如果应用最低支持4.0, 那么使用以下代码开启.

```
HttpResponseCache httpResponseCache = HttpResponseCache.install(new File(getCacheDir, "http"), CACHE_SIZE);
```

为应用选择一个合适的缓存大小. 如果只获取少量的数据, 可以选择几兆大小的缓存. 缓存对应用程序是私有的, 所以相对来说是比较安全的.

比如说`HTTP GET`请求, [可以看事例代码](https://github.com/suzeyu1992/AndroidProgrammingPushingTheLimits/blob/0d16eb99528aa7ed7bfc238e881222879ab7d95f/Network/app/src/main/java/com/szysky/note/network/HTTPGet.java)

**或者说上传文件**

比如说传送图片或者其他文件到服务器, 由于Java API并没有提供一个可以直接上传文件的的方法. 使用HTTP发送数据涉及使用`HTTP POST`发送`body`中的数据. 而body需要设置一些特殊的格式, 并且还要正确地设置`header`字段. 如下:

```
private static final long MAX_FIXED_SIZE = 5 * 1024 * 1024;
private static final String CRLF = "\r\n";
/**
*  使用HTTP POST往服务器发送文件
*/
public int postFileToURL(File file, String mimeType, URL url) throws IOException {

   DataOutputStream requestData = null;
   try {
       long fileSize = file.length();
       String fileName = file.getName();

       // 创建一个随机边界符字符串
       Random random = new Random();
       byte[] randomBytes = new byte[16];
       random.nextBytes(randomBytes);
       String boundary = Base64.encodeToString(randomBytes, Base64.NO_WRAP);

       // 配置请求设置
       HttpURLConnection uc = (HttpURLConnection) url.openConnection();
       uc.setUseCaches(false);
       uc.setDoOutput(true);           // 设置可以发送数据
       uc.setRequestMethod("POST");

       // 设置HTTP header
       uc.setRequestProperty("Connection", "Keep-Alive");
       uc.setRequestProperty("Cache-Control", "no-cache");
       uc.setRequestProperty("Content-Type", "multipart/form-data;boundary="+boundary);

       // 如果文件大于max_fixed_size, 使用分块流模式
       if (fileSize > MAX_FIXED_SIZE){
           uc.setChunkedStreamingMode(0);
       }else{
           uc.setFixedLengthStreamingMode((int)fileSize);
       }

       // 打开文件方便读取
       FileInputStream fileIn = new FileInputStream(file);
       // 打开服务器连接
       OutputStream out = uc.getOutputStream();
       requestData = new DataOutputStream(out);

       // 开始写数据
       // 首先写入第一个边界符
       requestData.writeBytes("--" +boundary + CRLF);
       // 让服务器知道文件名
       requestData.writeBytes("Content-Disposition: form-data; name=\""+ fileName + "\"; filename=\""+fileName + CRLF);
       // 文件的MIME类型
       requestData.writeBytes("Content-Type: "+mimeType + CRLF);

       // 循环读取本地文件, 并写入服务器
       int bytesRead;
       byte[] buffer = new byte[8 * 1024];
       while((bytesRead = fileIn.read(buffer)) != -1){
           requestData.write(buffer, 0, bytesRead);
       }

       // 写入边界字符串, 表明已到文件结尾
       requestData.writeBytes(CRLF);
       requestData.writeBytes("--" +boundary +"--"+ CRLF);
       requestData.flush();

       return uc.getResponseCode();

   } catch (IOException e) {
       e.printStackTrace();
   }finally {
       if (requestData != null){
           requestData.close();
       }
       return -1;
   }

}
```

上面代码, 需要要知道如何使用边界字符串告诉服务器文件的开始和结束位置. 另外上面还通过检查文件的大小而决定是`使用分块流模式(chunked streaming mode)`,还是`使用固定长度流模式` . 对于`分块流模式`参数0表示系统的默认大小, 这是大多数程序在该模式下的大小. 分块基本上意味着数据分部分发送数据, 每一部分都附有该块大小. 分块能更有效地使用内存, 并较少oom异常的分享, 然而, 使用固定长度的数据流模式通常更快, 但它需要更多的内存.

### OkHttp和SPDY

`HTTP`的一个大问题就是每个连接只允许一个请求和响应, 这迫使浏览器和其他客户端为了并行请求必须生成多个`套接字socket`连接. 虽然对于客户端连接问题就不那么大了, 但是如果是服务器端那么面临的状态就不同而语了. 在2009年, 谷歌开始着手更新`HTTP协议`来解决这些问题. 其结果就是`SPDY协议`, 它允许在一个套接字连接上发送多个`HTTP请求`. 该协议已成为下一代`HTTP`事实上的开放标准, 但它不会取代`HTTP`, 而是改良了如何通过网络请求和响应. 而`HTTP IETF`工作组日前已宣布即将开始`HTTP 2.0`的工作, 并使用`SPDY协议`作为起点.

如果同时开始客户端和服务端代码, 研究一下使用`SPDY`来代替常规的`HTTP/1.1`还是不错的, 因为`SPDY`能显著降低网络负载, 并能提高性能. 主流浏览器目前已经很好地支持`SPDY`, 并且已经有很多平台的实现版本, 启动就包括`Android`.

如果选择`SPDY`作为通信协议, 建议使用`OkHttp`, 它是`Square`公司开发的, 该库只是一个支持`SPDY`的新改进的`HTTP`客户端. 内部使用`HttpUrlConnection`接口.

当初始化`OkHttpClient`实例时, 它会初始化所有的东西, 比如连接池和响应缓存. 即使是普通的HTTP请求, 该实现也非常快, 使用OkHttp进行SPDY通信能显著提升网络调用的性能.

### Web Socket

运行标准的HTTP之上, 是HTTP的扩展. `Web Socket`允许在客户端和服务器端之前进行基于消息的异步通信. 首先客户端发送一个常规的`HTTP GET`请求, 该请求包含特殊的`HTTP请求头`, 表明客户端希望把连接升级为`Web Socket`连接.

如下当使用`Web Socket`时,客户端发送的请求示例:

```
GET /websocket HTTP/1.1
Host: myserver.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: xxxxxxxxxxxxxxxxxx==
Sec-WebSocket-Protocol: chat
Sec-WebSocket-Version: 13
Origin: http://myserver.com
```

如果接收客户端的请求, 下面是服务器对请求的响应:

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: xxxxxxxxxxxxxxxxxxx=
Sec-WebSocket-Protocol: chat
```

> 客户端的请求头中的值并不是对每一种情况都有效, 而应根据Web Socket协议规范来计算. 通常情况下, 如果使用现成的`Web Socket`通信库, 开发者就不需要考虑这些情况.

当`Web socket`连接建立后, 双方可以给对方发送异步消息. 通信的消息可以是文本, 或者二进制, 通常数据量是很小的. 如果需要传输大的文件, 最好还是使用标准的`Http`. `Web socket`用于发送符合相对较小的通知.

关于使用, 可以使用`Android`中标准的`Socket`类来实现自己的`Web Socket`客户端, 但也可以使用现有的第三方库. 比如由`Nathan Rajlich`为Java实现的WebSocket. 可以看[介绍](http://java-websocket.org/). 另外, 这个库还包含了一个服务器的实现.

## 网络和功耗

手机功耗, 一般手机中消耗第一应该是屏幕, 然后第二往往和网络流量有关.

智能手机的无线硬件如WiFi和蜂窝网络芯片都有内置省电功能, 他们能在网络流量不活跃时自动关闭连接, 并能把功耗降到一个非常低的水平. 当应用程序要发送数据或者等待接收输入数据和包时, 网络硬件将禁用省电模式, 以便能尽可能快速和有效地发送数据.

### 一般准则

在请求网络前, 首先考虑用户目前是否确实需要这些数据. 其次考虑是否需要全部数据, 或者只是数据的前十条而已, 如果服务器也可以进行gzip压缩, 要记得打开透明压缩, 并选择数据的数据格式, 例如json.

### 高效的网络轮询

网络轮询有几个缺点, 但是有时只能使用轮询来完成在线服务器的新数据的检查. 这时就别忘了`AlarmManager`这个API可以方便的进行轮询.

### 服务器端推送

减少网络调用次数最好的解决办法就是使用服务器端推送. 这样可以让服务器主动通知客户端有新的数据需要检索. 服务器端推送可以有很多种方式, 它可以不直接连接互联网(比如监听短信), 也可以是长期保持获取的常规TCP套接字连接.