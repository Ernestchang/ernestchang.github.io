title: 《Android 编程实战》Chap12_远程设备其余通信方式
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---
> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## Android中的连接技术

大多数`Android`设备都支持多种连接技术. 通常, 例如`USB`, `蓝牙`, `Wi-Fi`.

- `USB` 使用API通过USB进行原始串行通信, 或者使用谷歌专门为访问Android设备硬件配件定义的Android开放配件协议(Android Open Accessory Protocol, AOAP). AOAP是通过配件开发套件(Accessory Development Kit, ADK)支持的.
- `Bluetooth` Android设备都支持经典蓝牙配置文件(Classic Bluetooth Protocol), 它适合更耗电的操作, 比如视频流. `Android 4.3`开始支持蓝牙低功耗以及蓝牙智能(Bluetooth Smart)技术, 它能够和支持GATT配置的设备进行通信(如心脏检测器,计步器以及其他低功率配件)
- `Wi-Fi` 比如需要更多数据密集型通信的场景, `Android`支持三种Wi-Fi操作模式: `infrastructure(连接到一个接入点的的标准wifi)`, `网络共享(android设备充当其他设备wifi的接入点)`, `WiFi-Direct`这个模式比较有趣, 在一些新的设备可以和`infrastructure`模式进行并行工作. 允许应用程序建立对等的wifi网络, 而不需要专门的访问点.

## Android USB

Android中`USB`相关的API位于`android.hardware.usb`包. 如果需要usb外设的了解可以查看官网文档的usb/accessory相关介绍.

在`USB`的设计中, 会有一个设备充当**主机**. 除了其他功能, 主机还可以给所连接的设备供电, 这就是不需要给USB鼠标添加额外的电池, 以及可以使用笔记本上的USB端口给智能手机充电的原因.

`Android设备`也可以作为USB主机为外部设备充电, 这意味着可以把例如读卡器,指纹扫描,以及其他usb外设连接到手机设备上.

了解一下就好了如果需要更多, [参考资料](https://developer.android.com/guide/topics/connectivity/usb/index.html)

## 蓝牙低功耗

在`Android 4.3`开始支持蓝牙智能, 包括心率监视器, 活动跟踪器等蓝牙低功耗`BLE`设备的支持.

如果需要`蓝牙低功耗`比较感兴趣, 可以查看[博客的android蓝牙系列](http://szysky.com/2016/10/08/%E3%80%8AAndroid-%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98%E3%80%8B12-%E8%BF%9C%E7%A8%8B%E8%AE%BE%E5%A4%87%E5%85%B6%E4%BD%99%E9%80%9A%E4%BF%A1%E6%96%B9%E5%BC%8F/)

## Android Wi-Fi

`Wi-Fi`是Wi-Fi联盟管理的各种技术的统称. `Wi-Fi Direct`是运行在802.11n标准之上的额外技术. 使用该技术的设备不需要专门的连接点, 这点和蓝牙很相似, 不过`Wi-Fi Direct`使用高速的`Wi-Fi`进行通信.

但是, 即使设备都不再同一个`Wi-Fi`, 为了建立连接仍然需要发现它们. 发现意味着找到运行服务的设备的ip地址. Android已经内置了网络发现API, 支持标准的`Wi-Fi(infrastructure)和Wi-Fi Direct`, 可以让设备发现使用`DNS-SD`协议声明的服务.

### 服务发现

Android提供了标准的发现机制, 允许开发者宣布自己的服务以及发现本地网络上的服务. 该实现包括两个标准: `mDNS和DNS-SD`. `mDNS`是一个多播协议, 使用UDP组播协议宣布和发现主机. `DNS-SD`是一个服务发现协议, 用于宣布和发现运行在远程主机(通常限于本地网络)的服务. 可以通过`android.net.nsd`包以及`NsdManager`使用这些功能.

以下代码可以来声明一个设备中的服务:

```
/**
*  声明设备中标准Wi-Fi的服务
*/
private void announceService(){
   NsdManager nsdManager = (NsdManager) getSystemService(NSD_SERVICE);
   NsdServiceInfo nsdServiceInfo = new NsdServiceInfo();
   nsdServiceInfo.setPort(8081);
   nsdServiceInfo.setServiceName("wifi服务哦");
   nsdServiceInfo.setServiceType("_http._tcp.");

   nsdManager.registerService(nsdServiceInfo, NsdManager.PROTOCOL_DNS_SD, new NsdManager.RegistrationListener() {
       @Override
       public void onRegistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
           Log.e(TAG, "onRegistrationFailed: " );
       }

       @Override
       public void onUnregistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
           Log.e(TAG, "onUnregistrationFailed: ");
       }

       @Override
       public void onServiceRegistered(NsdServiceInfo serviceInfo) {
           Log.e(TAG, "onServiceRegistered: " );
       }

       @Override
       public void onServiceUnregistered(NsdServiceInfo serviceInfo) {
           Log.e(TAG, "onServiceUnregistered: " );
       }
   });
}
```

Note: 如果不给设置服务名, 那么会使用wifi网络中的设备IP地址. mDNS的服务类型必须是一个有效的类型. 再调用了注册方法后, `NsdManager`开始宣布在本地`Wi-Fi`上的服务, 当注册状态发生变化后会触发监听回调.

如果要发现一个服务, 使用同样的API实例:

```
/**
*  发现一个服务
*/
private void discoverService(){
   NsdManager nsdManager = (NsdManager) getSystemService(NSD_SERVICE);
   nsdManager.discoverServices("_http._tcp.", NsdManager.PROTOCOL_DNS_SD, new NsdManager.DiscoveryListener() {
       @Override
       public void onStartDiscoveryFailed(String serviceType, int errorCode) {
           Log.d(TAG, "onStartDiscoveryFailed: ");
       }

       @Override
       public void onStopDiscoveryFailed(String serviceType, int errorCode) {
           Log.d(TAG, "onStopDiscoveryFailed: ");
       }

       @Override
       public void onDiscoveryStarted(String serviceType) {
           Log.d(TAG, "onDiscoveryStarted: ");
       }

       @Override
       public void onDiscoveryStopped(String serviceType) {
           Log.d(TAG, "onDiscoveryStopped: ");
       }

       @Override
       public void onServiceFound(NsdServiceInfo serviceInfo) {
           Log.d(TAG, "onServiceFound");
           NsdManager nsdManager = (NsdManager) getSystemService(NSD_SERVICE);
           nsdManager.resolveService(serviceInfo, new NsdManager.ResolveListener() {
               @Override
               public void onResolveFailed(NsdServiceInfo serviceInfo, int errorCode) {

               }

               @Override
               public void onServiceResolved(NsdServiceInfo serviceInfo) {
                   Log.w(TAG, "主机: "+serviceInfo.getHost() +"      端口:"+serviceInfo.getPort() );
               }
           });

       }

       @Override
       public void onServiceLost(NsdServiceInfo serviceInfo) {
           Log.d(TAG, "onServiceLost: ");
       }
   });
}
```

注意, 需要使用服务类型来搜索服务, 一旦服务的状态发生变化(发现和丢失某些东西, 启动和停止发现服务)都会收到回调. 如果需要解析更详细的信息, 通过`NsdManager#resolverService()`方法来解析. 解析成功会回调函数.

通过`NsdManager`使用网络发现服务可以在不强制用户手动输入IP地址的情况下和本地设备进行通信. 当要创建共享数据的应用或者建立一个本地多人游戏时, 这是一个选择项.

### Wi-Fi Direct

`Wi-Fi Direct`是Wi-Fi联盟802.11标准的一部分, 允许在设备间进行高速的Wi-Fi通信, 而不需要专门的接入点. 它基本上是一个采用Wi-Fi技术的对等协议. 所有运行2.3及后续版本的设备都支持`Wi-Fi Direct`, 但是知道`Android 4.1`以及网络服务发现API的引入, 开发人员才真正对`Wi-Fi Direct`变得感兴趣.

在运行`Android 4.0`或更高的版本的设备上, 通常可以并行地运行`Wi-Fi Direct`, 这意味着设备可以同时支持`Wi-Fi Direct`以及普通的`Wi-Fi`.

主要API`WifiP2pManager` 来在一端设备创建并发布服务并监听变化, 另一端通过监听对等的设备并使用`WifiP2pServiceRequest`来搜索, 当搜索到可用符合的设备进行连接. 当两个设备建立连接, 服务端的注册的广播就会接收到通知. 并进行后续操作处理.

使用`WiFi Direct`最主要的原因是不需要现有的WIFI基础设施. 同时, 由于建立过程不需要额外的PIN码或者密码, 使用这种方式连接设备会很容易.