# 封包
> 在RTMP协议中信令和媒体数据都称之为Message，在网络中传输这些Message，为了区分它们肯定是要加一个Message header的，所以RTMP协议也有一个Message header，还有一个问题因为RTMP协议是基于TCP的，由于TCP的包长度是有限制的（一般来说不超过1500个字节），而RTMP的Message长度是有可能很大的，像一个视频帧的包可能会有几十甚至几千K，这个问题就必然有一个分片的问题，在RTMP协议中对应的说法就是chunk，每一个Message + header都是由一个和多个chunk组成的。

## Message
### 报文格式
![报文格式](./image/rtmp-message.jpeg)
不同种类的消息包含不同的Message Type ID，代表不同的功能。RTMP协议中一共规定了十多种消息类型，分别发挥着不同的作用。
- Message Type ID在1-7的消息用于协议控制，这些消息一般是RTMP协议自身管理要使用的消息，用户一般情况下无需操作其中的数据。
- Message Type ID为8，9的消息分别用于传输音频和视频数据。
- Message Type ID为15-20的消息用于发送AMF编码的命令，负责用户与服务器之间的交互，比如播放，暂停等等。

## Chunk
### 报文格式
![报文格式](./image/rtmp-chunk.jpeg)
