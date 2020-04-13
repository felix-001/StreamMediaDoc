# 封包
> 在RTMP协议中信令和媒体数据都称之为Message，在网络中传输这些Message，为了区分它们肯定是要加一个Message header的，所以RTMP协议也有一个Message header，还有一个问题因为RTMP协议是基于TCP的，由于TCP的包长度是有限制的（一般来说不超过1500个字节），而RTMP的Message长度是有可能很大的，像一个视频帧的包可能会有几十甚至几千K，这个问题就必然有一个分片的问题，在RTMP协议中对应的说法就是chunk，每一个Message + header都是由一个和多个chunk组成的。

## Message
### 报文格式
![报文格式](./image/rtmp-message.jpeg)
> 消息的报文格式同FLV文件的tag相同
- Message Type ID  
不同种类的消息包含不同的Message Type ID，代表不同的功能。RTMP协议中一共规定了十多种消息类型，分别发挥着不同的作用。
  - Message Type ID在1-7的消息用于协议控制，这些消息一般是RTMP协议自身管理要使用的消息，用户一般情况下无需操作其中的数据。
  - Message Type ID为8，9的消息分别用于传输音频和视频数据、18是脚本
  - Message Type ID为15-20的消息用于发送AMF编码的命令，负责用户与服务器之间的交互，比如播放，暂停等等。
-  Payload Length  
包长度
- Time Stamp  
时间戳
- Stream ID  
流ID
- Message Body  
音视频裸流(h264/aac/g711)

## Chunk
> 在网络上传输数据时，消息需要被拆分成较小的数据块，才适合在相应的网络环境上传输。RTMP协议中规定，消息在网络上传输时被拆分成消息块（Chunk）。消息块首部（Chunk Header）有三部分组成：用于标识本块的Chunk Basic Header，用于标识本块负载所属消息的Chunk Message Header，以及当时间戳溢出时才出现的Extended Timestamp。
### 报文格式
![报文格式](./image/rtmp-chunk.jpeg)

#### Chunk Basic Header  
格式

fmt(2bit) | cs id (6bit) 
---|---

- fmt  
fmt也称为head type，决定了`Chunk Message Header`的长度。  
- cs id
`Chunk Basic Header`的长度可能为1、2、3个byte，取决于`cs id`，这里的`cs id`是`chunk stream id`的简写。协议最大支持65597个`cs id `从3-65599. `cs id` 0、1、2为协议保留，0代表ID是64~319(第二个byte + 64). 1代表chunk stream ID为6-65599（（第三个byte）* 256 + 第二个byte + 64）(小端表示). 2代表该消息为低层的协议（在RTMP协议中控制信令的chunk stream ID都是2). 3-63的chunk stream ID就是该byte的值。

cs id为0时：
![报文格式](./image/rtmp-csid0.png)

cs id为1时：
![报文格式](./image/rtmp-csid1.png)

cs id为非0或1时：
![报文格式](./image/rtmp-csid-other.png)

#### Chunk Message Header

# RTMP流媒体播放过程
RTMP协议规定，播放一个流媒体有两个前提步骤：第一步，建立一个网络连接（NetConnection）；第二步，建立一个网络流（NetStream）。其中，网络连接代表服务器端应用程序和客户端之间基础的连通关系。网络流代表了发送多媒体数据的通道。服务器和客户端之间只能建立一个网络连接，但是基于该连接可以创建很多网络流。
播放一个RTMP协议的流媒体需要经过以下几个步骤：握手，建立连接，建立流，播放。RTMP连接都是以握手作为开始的。建立连接阶段用于建立客户端与服务器之间的“网络连接”；建立流阶段用于建立客户端与服务器之间的“网络流”；播放阶段用于传输视音频数据。


- RTMP官方协议
http://wwwimages.adobe.com/www.adobe.com/content/dam/Adobe/en/devnet/rtmp/pdf/rtmp_specification_1.0.pdf

- 开源项目 RTMP Dump
http://rtmpdump.mplayerhq.hu/
