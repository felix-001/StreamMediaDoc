# 介绍
WebRTC (Web Real-Time Communications) 是一项实时通讯技术，它允许网络应用或者站点，在不借助中间媒介的情况下，建立浏览器之间点对点（Peer-to-Peer）的连接，实现视频流和（或）音频流或者其他任意数据的传输。WebRTC包含的这些标准使用户在无需安装任何插件或者第三方的软件的情况下，创建点对点（Peer-to-Peer）的数据分享和电话会议成为可能。
# 概要
- 第一，通信双方需要先通过服务器交换一些信息
- 第二，完成信息交换后，通信双方将直接进行连接以传输音视频数据

# 组成
## 信令
webrtc规定了需要交换`sdp`，具体怎么交换，以及交换的方式，没有规定。只是定义了框架和接口，具体实现不限制。
- 信令服务器
- 传输协议
  - websocket
  - sip
  - http(srs)
- 流程
  - 收集信息
      - 需要的服务
        - STUN
          - 获取自己公网ip
          - p2p打洞
        - TURN - relay
        - ICE - 二者整合
    - 公网ip
    - NAT类型
      
  - 交换sdp、candidate
  - 建立p2p通道
  
## 流媒体
  - srtp

# 流程
![流程](./image/rtcFlow.png)
- 终端连接到 信令服务器，准备为两端进行信令交换。
- 发起端创建 PeerConnection，生成Offer 信令（SDP），通过 信令服务器 转发给另一端。
  SDP：描述建立音视频连接的一些属性，如音频的编码格式、视频的编码格式、是否接收/发送音视频等等。
- 响应端收到Offer 信令之后，生成Answer 信令（SDP）, 返回给发起端。
- 收到offer以后调用RTCPeerConnection.setRemoteDescription加入remote sdp
- WebRTC客户端先去连接ICE Server，也就是STUN Server或者TURN Server。
- 客户端连接STUN Server是为了测试出自己的NAT类型和获取到自己公网ip地址
- TURN Server包含了STUN Server的功能而且包含Relay中转功能。
- 当一个WebRTC客户端创建RTCPeerConnection并且设置好ICE和本地Audio Track、Video Track以后，会自动向ICE服务器发出测试然后得到ice candidate
- 交换完SDP之后，相互发送自己的Candidate信息。  Candidate：主要包含了相关方的IP信息，包括自身局域网的ip、公网ip、turn服务器ip、stun服务器ip等。
- 有了ip信息之后， 开始尝试进行 P2P打洞（打洞过程是框架实现的，如想知道打洞原理，可自行百度）。若打洞不成功，则会改用服务器转发。
- 无论是打洞还是转发，只要有一条路是成功的，那PeerConnection就算是成功建立了。接下来就可以进行音视频通话了。

# 实现流程
- addStream方法将getUserMedia方法中获取的流(stream)添加到RTCPeerConnection对象中，以进行传输
- onaddStream事件用来监听通道中新加入的流，通过e.stream获取
- onicecandidate事件用来寻找合适的ICE
- createOffer()是RTCPeerConnection对象自带的方法，用来创建offer，创建成功后调用setLocalDescription方法将localDescription设置为offer，localDescription即为我们需要发送给应答方的sdp
- sendOffer和sendCandidate方法是自定义方法，用来将数据发送给服务器

# 链接
- [webrtc api](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
