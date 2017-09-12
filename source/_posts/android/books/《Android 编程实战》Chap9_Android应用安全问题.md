title: 《Android 编程实战》Chap9_Android应用安全问题
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---
> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## Android安全的概念

`Android`具备一个先进的安全模型来保护应用数据和服务不被其他应用访问. 每个应用都有自己的唯一`ID`来提供最基本的保护. 每个应用都经过它唯一的密钥签名, 这种机制是`Android`框架中的安全模型基础. 此外只有当其他应用在清单文件中显式声明了正确权限后, `Android`的权限系统才会和他们共享特定的组件. 应用也可以定义权限, 比如只有使用同一个密钥签名的应用才能使用它们. 最后`Android`的API提供了各种方法来验证签名, 验证调用进程的用户ID和使用强加密方案.

### 签名和密钥

`Android`系统中运行的所有应用都要用密钥来签名, 包括`Android`系统本身.

可以用同一个密钥来对发布的所有应用进行签名, 但建议为各个应用单独创建一个密钥. 多个应用共享一个密钥通常是因为这些应用要直接访问彼此的数据, 或者设定权限时将保护等级改为`signature`

下面是一种为应用生成密钥的一个方式, 有一种比较好的方式是使用应用的包名作为别名传给`-alias`.

`$ keytool -genkey -v -keystore <keystore filename> -alias <alias for key> keyalg RSA -keysize 2048 -validity 10000`

在生成新密钥时, `keytool`会让你输入一个密码.

### Android权限

要在`Android`中使用特殊权限功能, 只要在清单文件中加入一个`uses-permission`标记即可. 它会告诉系统你的应用需要该项权限, 并在安装时通知用户这项需求.

`Android`中定义了五个保护等级: `常规normal`, `危险dangerous`, `同一签名signature`, `同一签名或系统signatureOrSystem`, `系统system`. 除非特殊指定, 默认等级一般为常规. 用来告知系统有应用要用到这个权限的函数. 只有将权限设为危险时, 它才会在用户安装(通常是通过Google Play Store)前提醒用户.

- `同一签名`保护等级要求应用使用跟定义该权限的应用相同的同一证书来签名. 这对设备制造商来说非常有用, 因为他们可以定义只有跟系统使用同一证书签名的应用才能使用的权限. 这样, 设备制造商就可以像他们使用受保护的系统服务的设备发布新应用.
- `同一签名或系统 以及 系统`这两个等级会告诉`Android`系统, 应用必须驻存在设备的系统分区上, 这样才能使用该权限. 这个功能最常见的例子是预装在系统分区上的Google应用. 这些应用可以使用许多常规应用无法企及的权限, 即使他们用的是Google的签名而不是设备制造商的.

> 也可以添加属性`android:permissionFlags="costsMoney"`它会告诉用户使用此权限的应用会产生费用, 例如要用到发短信的功能的应用. 只要应用提供了可能会给用户带来费用的API, 那就应该用带有此标记的权限来保护该API.

### 保护用户数据

如果要创建安全的数据文件, 不被其他应用访问, 可以在应用数据目录中存储文件. 而不是外部存储中.

如下:演示在应用的数据目录中对一个文件进行数据追加

```
public static void appendStringToPrivateFile(Context context, String data, String fileName){
    FileOutputStream out = context.openFileOutput(fileName, Context.MODE_APPEND | Context.MODE_PRIVATE);
    out.write(data.getBytes("UTF-8"));
    out.close();
}
```

这里使用了两个标志位:

- `MODE_APPEND` 要写入的数据都被追加到文件的末尾,
- `MODE_PRIVATE` 该文件只允许你的应用访问, 这个标识位也是默认标志位.

当然这只是一种比较安全的方式, 但是如果存储非常敏感的信息, 最好再对文件进行一些加密处理.

## 客户端数据加密

### Android的加密API

`Android`中的数据加密和解密API是基于Java SE的`javax.crypto`包中的API开始的. 实际的实现基于开源的`Bouncy Castle`加密API. 因此, 在开发`Android`应用时, 大多是使用Java SE的`javax.crypto`API就可以.

### 生成密钥

使用加密和解密函数时, 需要生成一个可根据用户输入(密码或其他安全方法)重新生成的安全且唯一的密钥.

下面的代码演示了如何为`AES算法`生成一个`SecretKey` `salt`是用于生成密钥的输入部分, 你需要记录下来. 在密码学中, `盐salt`是用做加密算法中单向函数输入的一段随机数据.

```
public static SecretKey generateKey(char[] password , byte[] salt) throws NoSuchAlgorithmException, InvalidKeySpecException {
   int iterations = 1000;
   int outputKeyLength = 128;

   SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA1");

   PBEKeySpec pbeKeySpec = new PBEKeySpec(password, salt, iterations, outputKeyLength);

   byte[] keyBytes = secretKeyFactory.generateSecret(pbeKeySpec).getEncoded();
   return new SecretKeySpec(keyBytes, "AES");
}
```

这个密钥在后面会使用

### 加密数据

要加密数据, 必须先生成用于加密的作为`Cipher`输入的盐和初始化向量. 下面的代码会通过`SecureRandom`类生成一个长度为8字节的盐. 注意: 不需要人工给`SecureRandom`喂种子, 系统会自动帮你处理. 创建一个初始化向量, 初始化`Cipher`, 然后将明文加密成字节队列. 有了密文数据之后, 可以使用`Base64`工具类从这些字节生成一个普通的String对象. 并把初始化向量和盐用同样的方式追加上去, 并通过一个`非Base64`字符来分开.

```
public static String encryptClearText(char[] password, String plainText)
       throws Exception {
   SecureRandom secureRandom = new SecureRandom();
   int saltLength = 8;
   byte[] salt = new byte[saltLength];
   secureRandom.nextBytes(salt);
   SecretKey secretKey = generateKey(password, salt);

   Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
   byte[] initVector = new byte[cipher.getBlockSize()];
   secureRandom.nextBytes(initVector);
   IvParameterSpec ivParameterSpec = new IvParameterSpec(initVector);
   cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivParameterSpec);
   byte[] cipherData = cipher.doFinal(plainText.getBytes("UTF-8"));
   return Base64.encodeToString(cipherData,
           Base64.NO_WRAP | Base64.NO_PADDING)
           + "]" + Base64.encodeToString(initVector,
           Base64.NO_WRAP | Base64.NO_PADDING)
           + "]" + Base64.encodeToString(salt,
           Base64.NO_WRAP | Base64.NO_PADDING);
}
```

方法返回的结果就是加密后的密文字符串. 解密的时候使用同样的规则解密即可

### 解密数据

和加密基本相似, 取出加密的的数据部分. 代码如下:

```
public static String decryptData(char[] password, String encodedData)
       throws Exception {
   String[] parts = encodedData.split("]");
   byte[] cipherData = Base64.decode(parts[0], Base64.DEFAULT);
   byte[] initVector = Base64.decode(parts[1], Base64.DEFAULT);
   byte[] salt = Base64.decode(parts[2], Base64.DEFAULT);

   Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
   IvParameterSpec ivParams = new IvParameterSpec(initVector);
   SecretKey secretKey = generateKey(password, salt);
   cipher.init(Cipher.DECRYPT_MODE, secretKey, ivParams);
   return new String(cipher.doFinal(cipherData), "UTF-8");
}
```

从上面方法可以了解到`Cipher`,`初始化向量`,`SecretKey`是如何通过输入的字符串重新生成的. 只要密码匹配, 就能够对数据进行解码和编码.

## 设备管理API

关于这部分, 更多偏向于系统级别的开发. 例如对远程安全加固功能, 当丢失设备通过短信,锁定设备等. 所以此处不做记录. 如果有兴趣可以查看API文档[Device Administration API](https://developer.android.com/guide/topics/admin/device-admin.html)