# 介绍
WebRTC (Web Real-Time Communications) 是一项实时通讯技术，它允许网络应用或者站点，在不借助中间媒介的情况下，建立浏览器之间点对点（Peer-to-Peer）的连接，实现视频流和（或）音频流或者其他任意数据的传输。WebRTC包含的这些标准使用户在无需安装任何插件或者第三方的软件的情况下，创建点对点（Peer-to-Peer）的数据分享和电话会议成为可能。
# 概要
- 第一，通信双方需要先通过服务器交换一些信息
- 第二，完成信息交换后，通信双方将直接进行连接以传输音视频数据

# 组成
## 信令
webrtc规定了需要交换`sdp`，具体怎么交换，以及交换的方式，没有规定。只是定义了框架和接口，具体实现不限制。
### 为什么信令不是由WebRTC定义的？
为了避免出现冗余，并最大限度地提高与已有技术的兼容性，WebRTC标准并没有规定信令方法和协议。JavaScript会话建立协议JSEP概述了这种方法：
WebRTC通话建立的思想是完全指定和控制媒体平面，但是尽可能将信令平面留给应用程序。其原理是，不同的应用程序可能更喜欢使用不同的协议，例如现有的SIP或Jingle呼叫信令协议，或者对于特定应用程序定制的东西，可能是针对新颖的用例。在这种方法中，需要交换的关键信息是多媒体会话描述，其指定了建立媒体平面所必需的传输和媒体配置信息。
JSEP的架构也避免了浏览器不得不保存状态，即作为一个信号状态机。如果信号数据在每次刷新页面的时候都会发生丢失，就会出现问题。相反，信号状态机可以保存在服务器上。

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
          - p2p打洞，nat类型
        - TURN - relay
        - ICE - 二者整合
    - 公网ip
    - NAT类型
      
  - 交换sdp、candidate
  - 建立p2p通道
  
## 流媒体
### DTLS
DTLS的主要用途，就是让通信双方协商密钥，用来对数据进行加解密。
- 通信双方：通过DTLS握手，协商生成一对密钥；
- 发送方：对数据进行加密；
- 发送方：通过UDP传输加密数据；
- 接收方：对加密数据进行解密；
### srtp
SRTP、SRTCP，分别在RTP、RTCP的基础上加了个S(Secure)，表示安全的意思

#### 音视频数据的发送过程：
- 通信双方：通过DTLS握手，协商生成一对密钥；
- 数据发送方：将音视频数据封装成RTP包，将控制数据封装成RTCP包；
- 数据发送方：利用加密密钥，对RTP包、RTCP包进行加密，生成SRTP包、SRTCP包；
- 数据发送方：通过UDP传输SRTP包、SRTCP包；

### SCTP
tream Control Transmission Protocol）：流控制传输协议。
RTP/RTCP主要用来传输音视频，是为了流媒体设计的。而对于自定义应用数据的传输，WebRTC中使用了SCTP协议。

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
# 调试工具
chrome://webrtc-internals 

# 弱网对抗技术

## 第一类：关键帧请求
主要包括SLI／PLI／FIR，作用是在关键帧丢失无法解码时，请求发送方重新生成并发送一个关键帧。这本质是一种重传，但是跟传输层的重传的区别是，它重传是最新生成的帧。

## 第二类：重传请求
主要包括RTX／NACK／RPSI
这个重传跟关键帧请求的区别是它可以要求任意帧进行重传

## 第三类：码率控制
主要包括REMB／TMMBR／TMMBN
TMMBR是Temporal Max Media Bitrate Request，表示临时最大码率请求。表明接收端当前带宽受限，告诉发送端控制码率。REMB是ReceiverEstimated Max Bitrate，接收端估计的最大码率。TMMBN是Temporal Max Media Bitrate Notification

## 第四类：冗余包
音视频FEC（Forward Error Correction，前向纠错）前向纠错技术来进行丢包恢复，由发送方进行FEC编码引入冗余包，接收方进行FEC解码并恢复丢失的数据包。

WebRTC中音频前向纠错与视频前向纠错的方式不同，音频前向纠错遵循RFC2198标准，视频前向纠错遵循RFC5109标准。两者有差异的原因是音频传输所占据的带宽比较小，即使增加1倍的带宽冗余，也不会造成太大的影响，而视频的一帧比较大，通常需要几个RTP数据包才能完全发送，因此不能像音频一样具有较大的冗余力度。

# srs前期webrtc demo的web代码
```
<!DOCTYPE html>
<html lang="zh-cmn-Hans">
<head>
<meta charset="utf-8">
</head>

<body>

rtc_media_player: <br>
<video id = "rtc_media_player" autoplay></video>

</body>

<script>

var PeerConnection = window.RTCPeerConnection || window.mozRTCPeerConnection || window.webkitRTCPeerConnection;
var SessionDescription = window.RTCSessionDescription || window.mozRTCSessionDescription || window.webkitRTCSessionDescription;

var url = "http://localhost:1985/api/v1/sdp/";

var method = "POST";
var shouldBeAsync = true;

var request = new XMLHttpRequest();

request.open(method, url, shouldBeAsync);
request.setRequestHeader("Content-Type", "application/json;charset=UTF-8");

var pc = new PeerConnection(null);

var constraints = {
    mandatory: {
        OfferToReceiveAudio: true,
        OfferToReceiveVideo: true
    }
};

var sendViewerOfferFn = function(desc) {
    console.log('sendViewerOfferFn:', desc);

    pc.setLocalDescription(desc);

    var sdp_json = {"sdp":desc.sdp, "app":"webrtc", "stream":"test"};
    request.send(JSON.stringify(sdp_json));
};

pc.createOffer(sendViewerOfferFn, 
    function(error) { 
        console.log('sendViewerOfferFn error:' + error); 
    }, 
    constraints
);

pc.onaddstream = function(event) {
    console.log('onaddstream');
    document.getElementById('rtc_media_player').srcObject = event.stream;
    rtc_media_player.load();
};

pc.onicecandidate = function(event) {
    console.log('onicecandidate');
};

pc.onconnectionstatechange = function(event) {
    console.log('onconnectionstatechange');
};

pc.onicegatheringstatechange = function(event) {
    console.log('onicegatheringstatechange');
};

pc.onsignalingstatechange = function(event) {
    console.log('onsignalingstatechange');
};

request.onerror = function(event) {
    console.log('http error');
};

request.onload = function () {
    console.log('onload,' , request.responseText);
    var json = JSON.parse(request.responseText);
    console.log('onmessage viewerResponse:', json.sdp);

    pc.setRemoteDescription(new SessionDescription({type:'answer', sdp:json.sdp}));
}

</script>

</html>

```
# 链接
- [webrtc api](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
- [srs rtc pr](https://github.com/ossrs/srs/pull/1638)
- [srs rtc issue](https://github.com/ossrs/srs/issues/307)
- [srs摄像头推流webrtc播放演示](https://www.cnblogs.com/dong1/p/12712229.html)
- [srs普通推流webrtcp播放](https://segmentfault.com/a/1190000024533847)
- [webrtc cdn实现](https://zhuanlan.zhihu.com/p/143974932?utm_source=wechat_session&utm_medium=social&s_r=0)
- [SCTP](https://tools.ietf.org/html/rfc4960)
- [DTLS](https://tools.ietf.org/html/rfc4347)
- [关于sfu级联]
- [srs webrtc回源](https://github.com/ossrs/srs/issues/2091)
      
