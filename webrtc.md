# 组成
## 信令
webrtc规定了需要交换`sdp`，具体怎么交换，以及交换的方式，没有规定
- 信令服务器
- 传输协议
  - websocket
  - sip
  - http(srs)
- 流程
  - 收集信息
      - 需要的服务
        - STUN
        - TURN
        - ICE
    - 公网ip
    - NAT类型
      
  - 交换sdp
  - 建立p2p通道
  
## 流媒体
  - srtp

# 流程
![流程](./image/rtcFlow.png)
