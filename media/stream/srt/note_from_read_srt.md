# SRT笔记

## 协议分析理解

SRT是基于UDP，并利用UDT协议思想改造而来，单纯根据协议的定义来看它只是一个传输层的协议，但是它的维护者通过libsrt和附带的一些文档为SRT应用到具体业务上给出了很多官方建议和规范。这里主要基于直播应用方向，对协议层和应用方面的关键功能进行分析。

### SRT连接

SRT协议设计上是面向连接的，连接的建立方式和TCP类似，定义客户端和服务端角色，服务端监听等待客户端主动发起连接，客户端发起连接握手通过后连接建立。

但libsrt还提供了`Rendezvous`模式（详细可以参考[《SRT_Alliance_Deployment_Guide》](resource/SRT_Alliance_Deployment_Guide_v1.3_2019_05_22.pdf) “Streaming using Rendezvous mode”章节）。简单理解为对等协商连接，两端都知晓对方的IP和端口的情况下同时发起连接协商，这种模式可以用在P2P模式穿透NAT建立连接，实际情况中这个工作模式很难有效的建立起连接，通常这个连接的建立还需要第三方服务（STUN TURN），其实在libsrt中提供UDP传输代理能更灵活有效的解决P2P链接的问题。这里主要分析直播应用场景，这个场景下还是以C/S模式为主。

在ffmpeg-srt模块和SRT协议文档中连接模式描述为`Caller/Listener`模式，这个模式和`Client/Server`角色划分是相同的概念，主要就是用于指定一个连接发起方和连接监听方。通常在直播应用中都是流服务器充当`Listener`，客户端充`Caller`，客户端发起连接，服务端接受连接，后续客户端可以请求拉流或者推流。

>在TCP中是通过端口和ip地址确定一个连接，在SRT中通过协议报文头中的`Socket ID`来标识一个连接。

### Stream ID

`Stream ID`是SRT应用中非常重要的一个组成部分，它的使用规范主要在[《SRT Access Control Guidelines》](libsrt/access_control.md)文档中说明，它类似于HTTP协议头，用于指定请求数据类型（文件或是其它），请求方法。例如客户端请求读取一个流，ID可以指定为`stream_id=#!::r=live1,t=stream,m=request`，客户端推流`stream_id=#!::r=live1,t=stream,m=publish`,使用键值对来表示选项，r选项表示资源名或路径，t选项表示数据类型，m选项表示数据请求方法。但是这个字段的使用是自由的完全由用户决定如何使用，在[SLS](sls/sls_overview.md)中是通过一个url格式的id来做相应的区分，例如推流`stream_id=uplive.sls.com/live1`，uplive.sls.com域表示推流，live1表示资源名。

在P2P连接中这个字段就没什么实际的用途了，可以根据约定自由使用。

## SRT实时性策略

SRT中的Live模式会调整拥塞控制算法，采用快速重传，尽可能快的将数据传输到对端，这样会造成对带宽的饥渴请求。为了保证数据实时性，SRT还有超时丢弃策略，发送端预测到包无法在设定的延迟时间内发送到对端就会直接丢弃，在接收端检测到包可能无法在指定延迟时间内到达也会直接丢弃，

接收端的丢弃策略通过ACK实现，直接将ACK中的数据报确认序号偏移，控制发送端跳过对应的数据。

### Socket Group

### 加密

### 流量控制

在[《SRT Live模式性能》](srt_live_perfomance.md)中讨论了高质量网络中提高数据传输稳定性，这里面还有一个问题就是libSRT是有带宽探测的，按理说数据重传应该只会在出现在数据才开始发送的一段时间内，但是实际上整个传输过程中一直存在数据重传。为什么？

## 参考资料

1. [《SLS简介》](sls/sls_overview.md)
2. 新一代直播传输协议SRT：<https://zhuanlan.zhihu.com/p/114436399>
