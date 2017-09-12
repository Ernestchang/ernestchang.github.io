title: 《Android 编程实战》Chap8_高级音频,视频及相机应用
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---
> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## 高级音频应用

`Android`音频API提供了一些高级功能, 开发者可以把他们集成到自己的应用中. 有了这些API, 就可以很容易的实现**VoIP网络电话**, 构建定制的流媒体音乐客户端, 实现低延迟的游戏音效. 此外, 还有提供文本到语音转换以及语音识别API, 用户可以直接使用音频和用户交互, 而不需要使用用户界面或者触控技术.

### 低延迟音频

`Android`有四个用来播放音频的API(算上MIDI那么就是5个)和三个用来录音的API. 接下里会简要介绍这些API.

------

**音频播放API**

- 音频播放默认使用`MediaPlayer`. 该类适合播放音乐或者视频, 既能播放流式资源(比如在线网络收音机), 也可以播放本地文件. 每个`MediaPlayer`都有一个关联的状态机, 需要在应用程序中跟踪这些状态. 开发者可以使用`MediaPlayer`类的API在自己的应用中嵌入音乐或者视频播放功能, 而无需额外处理或者考虑延迟问题.
- 第二种是`SoundPool`类, 它提供了低延迟的支持, 适合播放音效和其他比较短的音频, 比如可以使用`SoundPool`播放游戏声音. 但是, 它不支持音频流, 所以不适合那些需要实时音频流处理的应用如`VoIP`.
- 第三种是`AudioTrack`类, 它允许把音频流缓冲到硬件中, 支持低延迟播放, 甚至适合流媒体场景. `AudioTrack`通常能提供足够低的延迟, 可在`VoIP`或类似应用中使用.

下面代码展示如何在`VoIP`应用中使用`AudioTrack`

```
public class AudioTrackDemo {

    private final AudioTrack mAudioTrack;
    private final int mMinBufferSize;

    public AudioTrackDemo() {

        // 确定音频流的最小缓冲区和大小
        mMinBufferSize = AudioTrack.getMinBufferSize(16000, AudioFormat.CHANNEL_OUT_MONO, AudioFormat.ENCODING_PCM_16BIT);

        mAudioTrack = new AudioTrack(AudioManager.STREAM_VOICE_CALL,
                16000,
                AudioFormat.CHANNEL_OUT_MONO,
                AudioFormat.ENCODING_PCM_16BIT,
                mMinBufferSize * 2,
                AudioTrack.MODE_STREAM);
    }

    public void playPcmPacket(byte[] pcmData){
        if (mAudioTrack != null && mAudioTrack.getState() == AudioTrack.STATE_INITIALIZED){

            // 判断是否处在播放状态
            if (mAudioTrack.getPlaybackRate() != AudioTrack.PLAYSTATE_PLAYING){
                mAudioTrack.play();
            }

            mAudioTrack.write(pcmData, 0, pcmData.length);
        }
    }

    //  设置停止
    public void stopPlayback(){
        if (mAudioTrack != null){
            mAudioTrack.stop();
            mAudioTrack.release();
        }
    }

}
```

对这个类进行一下整理, 首先从构造函数开始. 最开始需要确定音频流的最小缓冲区大小. 要做到这一点, 需要知道采样率, 数据是单声道还是立体声, 以及是否使用8位或者16位`PCM`编码. 然后以采样率和采样大小作为参数调用`AudioTrack.getMinBufferSize()`, 该方法会以字节形式返回`AudioTrack`实例的最小缓冲区大小

接下来, 根据使用正确的参数创建`AudioTrack`实例, 第一个参数为音频的类型, 不同的应用使用不同的值. 对于`VoIP`这种应用来说使用`STREAM_VOICE_CALL`, 而对流媒体音乐应用则使用`STREAM_MUSIC`.

对于参数2,3,4会根据使用场景而有所不同. 这些参数跟别表示`采样率`, `立体声/单声道`, `采样大小`. 一般而言, 一个`VoIP`会使用`16kHz`的16位单声道, 而常规的音乐CD可能采用`44.1kHz`的16位立体声. 16位立体声采样率需要更大的缓冲区以及更多的数据传输, 但是音质会更好. 所有的`Android`设备都支持`PCM`以`8kHz`,`16kHz`,`44.1kHz`的采样率播放8位或者16位的立体声.

缓冲区大小参数应该是最小缓冲区大小的倍数, 实际取决于具体的需求, 有时网络延迟等因素也会影响缓冲区大小. **注意: 任何时候都应该避免使用空的缓冲区, 因为可能导致播放出现故障**

最后一个参数决定只发送一次音频数据`MODE_STATIC`还是连续发送数据流`MODE_STREAM`. 第一种情况需要一次发送整个音频剪辑. 对于持续发送音频流的情况, 可以发送任意大小块的`PCM数据`, 处理流媒体音乐或者VoIP通话时可能会使用这种方式.

------

**2. 录音API**

说道录制音频(也可能是视频), 首先要考虑的API是`MediaRecorder`. 和`MediaPlayer`类似, 需要在应用代码中跟踪`MediaRecorder`类的内部状况. 由于`MediaRecorder`只能把录音保存在文件中, 所以他不适合录制流媒体.

如果需要录制流媒体, 可以使用`AudioRecorder`, 它和之前描述的`AudioTrack`非常相似.

[链接代码演示了如何创建`AudioRecorder`实例录制16位单声道16kHz的音频采样](https://github.com/suzeyu1992/AndroidProgrammingPushingTheLimits/commit/0109285e016f34c5403195344987cee5a7b791e0)

其实和`AudioTrack`的创建过程非常给你相似, 在使用`VoIP`或者类似应用时可以很方便地把他们结合起来.

### OpenSL ES

前面说了3播放API和2个录制API. 还有最后一个API `OpenSL ES`, 它同时支持播放和录制. 该API是科纳斯组织(Khronos Group)的一个标准, 这个组织还负责`OpenGL API`

`OpenSL ES`提供了低级别的音频硬件访问和低延迟特性来处理音频播放和录制. 虽然`Android`中其他音频API都有方便的Java API, 但是`OpenSL ES`目前仅支持在`Android NDK`中使用本地C代码访问.

> `= =!!! 到这里能力受限, 无法实现出效果...所以此章抄书就此太监. 后续包括OpenGL 串联Surface用于视频MediaPlayer的渲染层或者相机处理把预览画面的流连接到纹理流, 并实施处理画面实现现实AR应用等.`