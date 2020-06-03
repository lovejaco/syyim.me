+++
title = "WebRTC 연결성 및 NAT 통과 기법"
date = "2020-06-03"
description = "WebRTC Connectivity and NAT Traversal"
tags = [
    "webrtc",
]
+++

이 게시물은 WebRTC 연결성 및 NAT 통과 기법이라는 주제의 사내 발표 내용을 정리한 글입니다.

## 들어가며

WebRTC 프로젝트를 시작하려 할때 처음 마주하게 되는것은 아마도 시그널링, SDP, STUN, TURN과 같은 생소한 용어들과 RTCPeerConnection, RTCSessionDescription, RTCIceCandidate 와 같은 WebRTC API를 다루는 코드입니다.

예제 코드를 작성하기에 앞서 WebRTC 앞에 놓인 인터넷 환경과 (p2p) 연결의 한계, 그리고 이를 극복하기 위한 방법과 원리에 대해 설명 하려고 합니다.

## What is WebRTC?

![](https://lh5.googleusercontent.com/RfgFLvUI1dpj19XnfQDia6CbyTkC6PRBs0ZhGm8RJPvTMbKcb6tiBqbKbPFW-oVoTx8j4KbLmanpj0yuV8_-yJOT_JM_z0P_MdQpohWco_s7RFfD690CqwngnQjO_M9BvdqiAwu9Prc)

[WebRTC(Web Realtime Communications)](https://ko.wikipedia.org/wiki/WebRTC)는 웹 브라우저 간에 플러그인 없이 직접적으로(p2p) 통신할 수 있도록 설계된 API 입니다. W3C 에서 제시된 초안이며, 음성 통화, 영상 통화, P2P 파일 공유 등으로 활용될 수 있습니다

## WebSocket vs WebRTC

WebSocket과 WebRTC 모두 양 방향(bi-directional) 및 [전이중 통신: full-duplex](https://ko.wikipedia.org/wiki/%EC%9D%B4%EC%A4%91%ED%86%B5%EC%8B%A0) 을 지원 한다는 공통점을 가지고 있습니다. 간단히 차이점에 대해 비교합니다.

### WebSocket

- A Client Server Archetecture
- Runs Over TCP

### WebRTC

- allowing direct [peer-to-peer](https://ko.wikipedia.org/wiki/P2P) communication,
- Runs Over UDP

![](https://bloggeek.me/wp-content/uploads/2019/05/201905-websocket-vs-datachannel.jpg)

### Media over a WebSocket

WebRTC DataChannel과 마찬가지로 WebSocket 역시 “arraybuffer” 또는 “blob”과 같은 바이너리 형식을 지원하지만 [Real-time communication](https://en.wikipedia.org/wiki/Real-time_communication) 데이터를 처리하기에는 적절하지 않습니다.

대부분의 경우 실시간 미디어는 WebRTC 또는 RTSP, RTMP, HLS 등과 같은 다른 프로토콜을 통해 전송됩니다.

### WebRTC: Media Transport and Use of RTP

![](https://hpbn.co/assets/diagrams/f91164cbbb944d8986c90a1e93afcd82.svg)

WebRTC는 RTP 프로토콜을 이용하여 미디어를 교환합니다. [Web Real-Time Communication (WebRTC: Media Transport and Use of RTP](https://tools.ietf.org/html/draft-ietf-rtcweb-rtp-usage-26)는 WebRTC에서 실시간 전송 프로토콜 (RTP)이 사용되는 방법과 RTP 기능에 대한 요구 사항을 제공합니다.

### 실시간 전송 프로토콜 (RTP)

[실시간 전송 프로토콜(Real-time Transport Protocol, RTP)](https://ko.wikipedia.org/wiki/%EC%8B%A4%EC%8B%9C%EA%B0%84_%EC%A0%84%EC%86%A1_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)은 IP 네트워크 상에서 오디오와 비디오를 전달하기 위한 통신 프로토콜 입니다. RTP는 전화, WebRTC, 텔레비전 서비스, 웹 기반 푸시 투 토크 기능을 포함한 화상 통화 분야 등의 스트리밍 미디어를 수반하는 통신, 엔터테인먼트 시스템에 사용됩니다.

![](https://lh6.googleusercontent.com/XCvjkQoMEPNWn8ama4iiO92bvcChyzGFZijcRDFu37m0GFMfNZMdijZHXqT3F7_OzDAmVfzvPM_TKFbhjgBku87rJfXxi4bhafX3jCMdOoQk-YTAm84_wdjVRx8ROkaD_1zz_lwvVm4)

RTP는 일반적으로 사용자 데이터그램 프로토콜(UDP)로 동작하며, RTP는 RTCP(RTP Control Protocol)와 결합하여 사용됩니다.

## WebRTC 연결성 (connectivity)

[WebRTC connectivity - MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Connectivity)에서 다양한 WebRTC 관련 프로토콜이 피어 간의 연결을 만들고 데이터 및 미디어를 전송하기 위해 상호 작용하는 방법을 설명합니다.

그전에, 이런 절차가 왜? 필요한지에 관해 알아보려 합니다.

![](https://imagescdn.gettyimagesbank.com/500/18/288/575/0/1030419304.jpg)

온라인에서 모르는 누군가와 (p2p) 연결을 시도하는 것은 오프라인에서 연락처를 교환하고 서로 통화를 시도하는 과정과 유사합니다.

### 온라인 식별

전화번호를 이용하여 상대방을 식별하는 것과 마찬가지로 인터넷 환경에서는 IP 주소를 이용합니다.

#### 공인 IP, 사설 IP, 고정 IP, 유동 IP

피어간 연결을 위해서 상대의 IP 주소를 알아야 합니다.

- 공인 IP: 관리 기관에서 부여한 전세계에서 **유일한** IP**주소**
- 사설 IP: 임의로 부여한 IP, 자신의 네트워크 안에서만 **유일한** IP**주소**
- 고정 IP: IP를 반납하기 전까지는 다른 장비에 부여할수 없는 IP
- 유동 IP: 사용할 때 남아있는 IP 중에서 부여하는 IP

### Signaling

누군가 나의 전화번호를 이용하여 연결을 시도하면 내 전화기의 전화벨이 울리고 상대방은 대기합니다. 저는 이 연결을 수락할 수도 있고 거부할 수도 있습니다. 때로는 시간 초과로 연결이 실패할 수도 있습니다.
이처럼 통신 세션을 설정, 제어, 종료하는 프로세스를 시그널링이라고 부릅니다.

p2p 연결은 ip:port를 open 하거나 listen 함으로서 동작합니다. 따라서 사전에 상호 간 연결 설정을 교환하고 동의해야 합니다.

### NAT/Firewall Problem

IP 주소를 알고 있고, 연결할 준비가 되었다고 해서 늘 연결에 성공하는 것은 아닙니다. 공유기와 같은 라우터 장비 내부에 있는 디바이스는 p2p 연결에 어려움을 겪습니다.

## NAT (Network Address Translation)

[네트워크 주소 변환(영어: network address translation, 줄여서 NAT)](https://ko.wikipedia.org/wiki/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC_%EC%A3%BC%EC%86%8C_%EB%B3%80%ED%99%98)은 컴퓨터 네트워킹에서 쓰이는 용어로서, IP 패킷의 TCP/UDP 포트 숫자와 소스 및 목적지의 IP 주소 등을 재기록하면서 라우터를 통해 네트워크 트래픽을 주고 받는 기술을 말합니다.

가정용 라우터(공유기)와 같이 NAT(Network Address Translation)는 라우터의 일부로서, 그리고 통합된 방화벽의 일부로서 한번에 두가지 문제를 처리합니다.

![](https://lh3.googleusercontent.com/W44dghqO284h-65OyhXeKgrqpsrKy0g4OdXE6DCI6ZYd6zLXUGv75bqyPA1iGcNI3Siqx1RJ1jQqy_o6-fYRxiw9Zx7wYv90hDuCl3QhXZKMCjNe--kqt5J1oIQU0V-i_Wl4V_r_GMU)

인터넷 공인(Publid) IP 주소는 한정되어 있기 때문에 NAT는 사설(Private) IP를 사용하면서 이를 공인 IP로 상호변환할수 있도록 지원 합니다.

![](https://ltwus2ix28x10gixx34jeigv-wpengine.netdna-ssl.com/wp-content/uploads/2013/08/NAT-post-image-1.png)

공개된 인터넷과 사설망 사이에 방화벽을 설치하여 외부 공격으로부터 네트워크를 보호하는 수단으로 사용됩니다.

### 일반적인 NAT 시나리오

![](https://lh5.googleusercontent.com/UEU4ipyrC9ObBe6P4WstA8klYEpcUTA-sCrO8lzCNYajODZMhLBP3-GTzjl3B0OWB4HuPsSw8Vjj9qAWqZgAR_Hd-YSE0xRcOgyOYCYzQ3rbzXvHz26hiWINIt3WLmDS2hrgd48-J1o)

대부분의 방화벽은 인터넷의 특정 패킷이 NAT를 통과하도록 허용하지만, 호스트가 먼저 방화벽을 통해 패킷을 내보낸 경우에만 가능합니다. 본질적으로 통신은 LAN의 호스트에 의해 시작되어야 합니다.

모든 패킷에는 5개의 튜플로 이루어진 TCP/IP 정보로 구성되어 있습니다.

> source [ip:port], destination[ip:port], transport protocol

패킷이 NAT/방화벽 장치를 통과하면 소스 IP 및 소스 포트가 새로운 소스 IP 및 소스 포트로 변경됩니다. 그런 다음 “내부” 소스 IP 및 소스 포트와 “외부” 소스 IP 및 소스 포트 사이의 바인딩(이를 NAT 바인딩이라고 하며) 정보를 테이블에 저장합니다.

![](https://ltwus2ix28x10gixx34jeigv-wpengine.netdna-ssl.com/wp-content/uploads/2013/08/NAT-post-image-3.png)

인터넷에 있는 호스트는 변경된 “외부” IP:Port만 알 수 있습니다. 호스트가 이 “외부” IP:Port를 확인하고 이 주소로 패킷을 보내면 먼저 NAT/방화벽 장치로 도착합니다. NAT/방화벽 장치는 NAT 테이블을 조회한 다음, NAT 바인딩을 기반으로 대상(target) IP:Port를 변경하고 패킷을 내부 호스트로 전달합니다.

### NATs and Peer-to-Peer

![](https://ltwus2ix28x10gixx34jeigv-wpengine.netdna-ssl.com/wp-content/uploads/2013/08/NAT-post-image-4.png)

클라이언트 - 서버 간 통신은 NAT/방화벽을 잘 통화하지만 p2p 연결에는 어려움이 따릅니다. 각 엔드포인트 사이에 NAT/방화벽 장치가 존재할 가능성이 있기 때문입니다.

![](https://ltwus2ix28x10gixx34jeigv-wpengine.netdna-ssl.com/wp-content/uploads/2013/08/NAT-post-image-5.png)

엔드포인트는 자신과 통신하려는 피어 사이에 어떤 종류의 네트워크 토폴로지(링크, 노드의 요소들)가 있는지 알 수있는 방법이 없습니다. 따라서, 로컬 호스트에서 전송한 패킷이 방화벽에 의해 차단 되는 것을 알지 못하며, 추가적으로 엔드포인트에 의해 제출된 주소는 그 사이에 있는 NAT 때문에 다른 피어가 연결할 수 조차 없습니다.

## NAT를 넘어 P2P 연결하기

1. Relaying
2. Connection Reversal
3. Hole Punching
   1. UDP
   2. TCP

### Relaying

Relaying 방법은 사설 IP 주소를 가지는 두 단말 간에 직접 통신을 할 수 없다면 공인 IP 주소를 가지는 외부 서버를 통해 P2P 데이터 패킷을 Relay하자는 개념입니다.

### Connection Reversal

![](https://bford.info/pub/net/p2pnat/img7.png)

두 호스트 모두 잘 알려진 서버에 연결되어 있고 피어 중 하나만 NAT 뒤에있을 때 통신을 가능하게하기 위해 연결 반전이라고 알려진 간단하지만 제한된 기술을 사용합니다. B가 A에 대한 연결을 시작하려는 경우 A에 대한 모든 직접 연결 시도는 A의 NAT에 의해 차단됩니다. 대신 서버 S를 통해 A로 연결 요청을 릴레이하여 A에게 B로의 "역방향" 연결을 다시 시도하도록 요청합니다. 이 아이디어는 기술적인 한계에도 불구하고, 다음에 설명하는 일반적인 홀 펀칭 기술의 기초가 됩니다.

### Hole Puncing

P2P 통신을 목적으로 Routing Table 을 작성하기 위해 사전에 상대방과 패킷을
주고받게 하여 각자의 공유기에 Routing Table을 작성하는 것을 홀 펀칭이라고 합니다.

![](https://www.researchgate.net/profile/Wilhelm_Hasselbring/publication/250030374/figure/fig2/AS:298352580284420@1448144206583/Simplified-hole-punching-example.png)

흔히 [UDP 홀펀칭](https://en.wikipedia.org/wiki/UDP_hole_punching) 으로 널리 알려졌지만, [TCP에 대해서도 적용할 수 있습니다.](https://en.wikipedia.org/wiki/TCP_hole_punching)

![](https://bford.info/pub/net/p2pnat/img8.png)

자세한 설명은 다음의 링크에서 확인할 수 있습니다.

- [– Bryan Ford’s Home Page](https://bford.info/pub/net/p2pnat/)
- [P2P와 NAT: NAT 통과 기법 소개 (RFC 5128) - 2편: UDP Hole Punching | NETMANIAS](https://www.netmanias.com/ko/post/blog/6263/nat-network-protocol-p2p/p2p-nat-nat-traversal-technic-rfc-5128-part-2-udp-hole-punching)

## WebRTC tools for NAT/Firewall Traversal

WebRTC는 NAT 통과 기법을 위해 다음의 네트워크 표준을 이용합니다.
이 표준들은 위에서 설명한 P2P 연결 방법에 대한 구체적이고 표준화된 연결 방법입니다.

### Interactive Connectivity Establishment (ICE) – RFC 5245

#### 시그널링

WebRTC에서 서로 다른 네트워크에 있는 2개의 디바이스들을 서로 연결하기 위해서는, 각 디바이스들의 위치를 발견하는 방법과 미디어 포맷 협의가 필요합니다. 이 프로세스를 **시그널링** **signaling**이라 부르고 상호 간에 동의된 서버에 연결합니다. 이 서버는 각 디바이스들이 **negotiation**(협상) 메시지들을 교환할 수 있도록 중개합니다.

#### The signaling server

WebRTC는 시그널링 정보에 관한 transport 메커니즘을 제시하지 않습니다. 두 피어들 사이에서 정보를 전달해 줄 수 있는 것이라면 어떤 것이든 상관 없습니다.

- XMLHttpRequest
- long polling
- WebSocket
- MQTT Over WebSocket

> 시그널링 서버는 SDP를 전달하지만 데이터 내용은 몰라도 됩니다. 메시지의 내용들은 각 피어에서 생성되고 시그널링 서버를 통해 상대편으로 전송 됩니다.

#### Session Description Protocol (SDP)

[세션 기술 프로토콜 - SDP](https://ko.wikipedia.org/wiki/%EC%84%B8%EC%85%98_%EA%B8%B0%EC%88%A0_%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C)은 스트리밍 미디어의 초기화 인수를 기술하기 위한 포맷으로. 이 규격은 IETF의 RFC 4566로 규정되어 있습니다.

SDP는 해상도, 형식, 코덱, 암호화등의 멀티미디어 컨텐츠의 연결을 기술(Description)하기 위한 표준 입니다. 기술적으로는, 실제 프로토콜이 아니라 장치간에 미디어를 공유하는 연결을 설명하는데 사용되는 데이터 형식입니다.

### ICE candidates

피어는 (위에서 설명한 SDP로) 미디어에 대한 정보를 교환 할뿐만 아니라 네트워크 연결에 대한 정보를 교환해야합니다. 이를 ICE candidates라고 하며 피어가(직접적으로 또는 TURN 서버를 통해 경유하여) 사용 가능한 방법을 자세히 설명합니다.

> 예를 들어 연락처를 교환할때 보통은 휴대폰 번호만을 주고받는것이 일반적이지만, 필요에 따라 집 또는 직장 번호를 교환하기도 합니다. 이 처럼 통화 가능한 연락처의 목록을 ICE candidates (ip, port, transport) 라고 부르며, 피어가 능동적으로 이 후보들을 설명합니다.

일반적으로 각각의 피어는 가장 좋은 후보에서 나쁜 후보 순서로 제안하고 연결을 위해 나아가게 됩니다. UDP 후보가(더 빠르고, 미디어 스트림이 비교적 쉽게 인터럽트로부터 회복할 수 있기 때문에) 이상적이지만, ICE 표준은 TCP 후보 역시 허용합니다.

#### RTCIceCandidateType

- host
  호스트 후보는 호스트의 인터페이스(VPN 인터페이스를 포함하여 물리적 또는 가상)에 연결된 실제 IP 주소 입니다.
- srflx
  server reflexive candidate는 STUN / TURN 서버에 의해 식별된 IP 주소로, 클라이언트의 공인 IP 주소 입니다.
- prflx
  peer reflexive candidate는 각 피어에 의해 식별된 IP 주소, 각 피어간 STUN Check를 통하여 식별된 IP 주소입니다.
- relay
  relay candidate는 server reflexive candidate( "srflx")처럼 생성되지만 STUN 대신 TURN을 사용합니다.

### Session Traversal Utilities for NAT (STUN) – RFC 5389

![](https://d20hvw4zeymqbm.cloudfront.net/wp-content/uploads/2013/09/stun.png)

[Session Traversal Utilities for NAT (STUN)](http://en.wikipedia.org/wiki/STUN)은 P2P 통신을 위해 호스트가 NAT의 존재 유무 및 NAT 타입을 식별(discover)하고 또한 NAT에 의해 변경되는 외부 IP 주소 및 Port 값을 발견하기 위한 네트워크 프로토콜입니다.

STUN은 클라이언트-서버 프로토콜입니다. STUN 클라이언트는 공용 IP 및 포트를 발견하기 위해 STUN 서버에 요청을 보내고 STUN 서버는 응답을 리턴합니다. 요청에는 일반적으로 UDP를 통해 전송되는 Binding Request와 TCP 와 TLS (보안 통신) 를 통해 전송되는 Shared Secret Request입니다.

stun message type

- 0x0001 : Binding Request
- 0x0101 : Binding Response
- 0x0111 : Binding Error Response
- 0x0002 : Shared Secret Request
- 0x0102 : Shared Secret Response
- 0x0112 : Shared Secret Error Response

### Traversal Using Relay NAT (TURN) – RFC 5766

위에서 Relaying에 대해 설명한것과 마찬가지로 WebRTC 역시 공인 IP 주소를 가지는 외부 서버를 통해 P2P 데이터 패킷을 Relay 하는데 이를 TURN이라고 부릅니다.

![](https://img.favpng.com/7/5/3/traversal-using-relays-around-nat-stun-nat-traversal-computer-servers-webrtc-png-favpng-mQRBjt11JcX3SKAL9YixwsCLZ.jpg)

몇몇의 라우터들은 Symmetric NAT이라고 불리우는 NAT를 채용하고 있습니다. [Traversal Using Relays around NAT (TURN)](http://en.wikipedia.org/wiki/TURN) 은 TURN 서버와 연결하고 모든 정보를 서버에 전달하는 것으로 Symmetric NAT 제한을 우회하는 것을 의미합니다. 이것은 명백히 오버헤드가 발생하므로 이 방법은 다른 대안이 없을 경우만 사용하게 됩니다.

![](https://www.frozenmountain.com/hs-fs/hubfs/4%20-%20blog%20images/Turn%20Stun%20chart.png?width=1490&name=Turn%20Stun%20chart.png)

## WebRTC Call Flow (호출 흐름) - API

이제 WebRTC 의 연결 흐름에 대해 알아 보겠습니다.

### Signalling

```js
const configs = {
  iceServers: [
    {
      urls: "stun:stun.l.google.com:19302",
    },
    {
      urls: "turn:192.158.29.39:3478?transport=udp",
      credential: "JZEOEt2V3Qb0y27GRntt2u2PAYA=",
      username: "28224511:1379330808",
    },
    {
      urls: "turn:192.158.29.39:3478?transport=tcp",
      credential: "JZEOEt2V3Qb0y27GRntt2u2PAYA=",
      username: "28224511:1379330808",
    },
  ],
};

const peerConnectionOptions = {
  optional: [
    {
      DtlsSrtpKeyAgreement: true,
    },
  ],
};

const myPeerConnection = new RTCPeerConnection(configs, peerConnectionOptions);
```

![](https://www.html5rocks.com/ko/tutorials/webrtc/infrastructure/turn.png)

RTCPeerConnection 설정에 따라 STUN/TURN 서버의 구성이 변경될 수 있습니다.

#### Offer / Answer

![](https://hpbn.co/assets/diagrams/69aa329ffbfae6fd0446de77623c93fb.svg)

#### Offer SDP

![](https://lh4.googleusercontent.com/5MUbymu8G_iRPSn_ks-eLO8OH7DP6NW_rXMtp9BgWY29T4LZi-JVq2YOXuWey5lmEuCqihms-xzEi2RFUb1SblhtyxbhSyA7cOaAIwrrnolDBpTcQatVPGNBPIjEZr0XtFLXrISdQNc)

```js
myPeerConnection.createOffer().then(function (offer) {
  return myPeerConnection.setLocalDescription(new RTCSessionDescription(offer));
});
```

#### Answer SDP

![](https://lh6.googleusercontent.com/xQc7L7-RTOyNkVTIIjwR6fF_iPziPsyw9Fx3l_GGmN8rlh4l-11ylMHo-X0jg4b0L2k0R_cL9-HoKRGveaoq-v4HegwWqyB8jR5qpNB2Fl1m8MUXZXZmBzzZIgbK9oytd-BBUvgfatg)

```js
myPeerConnection
  .setRemoteDescription(new RTCSessionDescription(description))
  .then(function () {
    return createMyStream();
  });
```

시그널 서버로부터 전달받은 상대의 remoteDescription을 등록합니다.

![](https://cdn-images-1.medium.com/max/800/1*X4iOI4qIKwoC8oK7AFdr6w.jpeg)

1. A: RTCPeerConnection.createOffer() 호출하여 제안(Offer) 형식(sdp 포맷)을 생성
2. A: peerConnection.setLocalDescription(sdp) 생성된 제안을 peerConnection 객체에 등록
3. A: send To SignalServer(sdp): **sendOffer** 시그널 서버로 제안(SDP) 전송
4. B: receive From SignalServer(sdp): **receiveOffer** B가 시그널 서버로부터 제안(SDP)을 수신
5. B: peerConnection.setRemoteDescription(sdp) B가 A의 제안(SDP)을 peerConnection 객체에 등록
6. B: RTCPeerConnection.createAnswer(): B가 수락(Answer) 형식(sdp 포맷)을 생성
7. B: peerConnection.setLocalDescription(sdp) 생성된 수락을 peerConnection 객체에 등록
8. B: send To SignalServer(sdp): **sendAnswer** 시그널 서버로 수락(SDP) 전송
9. A: receive From SignalServer(sdp): **receiveAnswer** A가 시그널 서버로 부터 B의 수락 형식을 수신
10. A: peerConnection.setRemoteDescription(sdp) A가 B의 수락(SDP)을 peerConnection 객체에 등록

### Interactive Connectivity Establishment (ICE)

대화식 연결 설정

#### Ice gathering

setLocalDescription 호출이 완료되면 피어는 자신의 ice 후보를 수집합니다.

```js
pc.onicecandidate = function (event) {
  if (event.candidate) {
    // event.candidate가 존재하면 원격 유저에게 candidate를 전달합니다.
  } else {
    // 모든 ICE candidate가 원격 유저에게 전달된 조건에서 실행됩니다.
    // candidate = null
  }
};
```

#### Exchange Ice Candidates

각 피어는 식별된 자신의 ice candidates를 SDP에 패킹하거나 독립적으로 전송하도록 선택할 수 있습니다.

- SDP에 포함  
  ICE 후보가 SDP 제안/응답에 포함될 수 있습니다. 이렇게 하려면 ICE 수집 프로세스가 완료 될 때까지 기다린 다음 SDP 제안 / 응답을 보내십시오.

- TrickleICE  
  **ICE trickling** 은 초기 offer 혹은 answer를 다른 유저에게 이미 전달을 했음에도 계속해서 candidate를 보내는 과정을 뜻합니다. 이 속성은 [RTCPeerConnection.setRemoteDescription()](https://developer.mozilla.org/ko/docs/Web/API/RTCPeerConnection/setRemoteDescription)가 호출된 후에만 설정됩니다. TrickleICE가 구현 된 경우 (Chrome 및 Firefox) 첫 번째 양호한 ICE 후보가 발견되면 연결이 시작됩니다.

Trickle ICE는 시그널 채널을 통해 더 많은 트래픽을 생성하지만 p2p 연결을 시작하는데 필요한 시간이 크게 향상 될 수 있습니다.

```js
signalingChannel.onmessage = (receivedString) => {
  const message = JSON.parse(receivedString);
  if (message.ice) {
    pc.addIceCandidate(message.ice).catch((e) => {
      console.log("Failure during addIceCandidate(): " + e.name);
    });
  } else {
    // handle other things you might be signaling, like sdp
  }
};
```

Websocket등을 이용하여 시그널 서버로 전송하고, 전송받은 후보를 등록

##### STUN Check / DTLS / RTP / SCTP

![](https://lh4.googleusercontent.com/Ag31x7cJXPixbM1pwasQil5zHDjJwhRqAe56CHUhOUgYalTqdL4_sRCJawEOF85m1KS2kzNkSY8mZlQuc9QXFTNLv8ml_Xci60rxwviDERQ-B9f0K5vGt79699ce88YtxshQoU68X48)

전달받은 상대방의 연결 후보(IP 주소)를 이용하여 연결을 시도 합니다.

- STUN  
  STUN 프로토콜을 이용하여 확인 합니다.
  서로 동시에 연결을 시도하는 경우 STUN 에러가 발생하기도 합니다. 그러나 프로토콜 흐름에 따라 한쪽에서 기다리고 다른 한쪽에서 연결을 시도하여 확인 합니다.
- DTLS  
  STUN 연결이 확인되면 DTLS(보안 연결)을 이용하여 RTP 스트림에 사용되는 키를 설정합니다. [RFC 5764 - Datagram Transport Layer Security (DTLS) Extension to Establish Keys for the Secure Real-time Transport Protocol (SRTP)](https://tools.ietf.org/html/rfc5764)
- RTCP  
  [RTCP](https://ko.wikipedia.org/wiki/RTCP) 는 RTP 세션의 대역 외(out-of-band) 통계 및 제어 정보를 제공한다. 멀티미디어 데이터의 전달, 패키징 시에 RTP와 함께 사용하지만 RTCP가 직접 미디어 데이터를 전송하지는 않는다.
- RTP

감사합니다.

[참고 링크]

- [An Intro to WebRTC’s NAT/Firewall Problem - webrtcHacks](https://webrtchacks.com/an-intro-to-webrtcs-natfirewall-problem/)
- [RTCPeerConnection - Web API | MDN](https://developer.mozilla.org/ko/docs/Web/API/RTCPeerConnection)
- [WebRTC connectivity - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Connectivity)
- [Signaling and video calling - Web API | MDN](https://developer.mozilla.org/ko/docs/Web/API/WebRTC_API/Signaling_and_video_calling)
- [WebRTC NAT Traversal Methods: A Case for Embedded TURN](https://www.frozenmountain.com/developers/blog/webrtc-nat-traversal-methods-a-case-for-embedded-turn)
- [WebRTC vs WebSockets • BlogGeek.me](https://bloggeek.me/webrtc-vs-websockets/)
- [websockets vs webrtc | 7 Most Amazing Comparisons To Learn](https://www.educba.com/websockets-vs-webrtc/)
- [Browser APIs and Protocols: WebRTC - High Performance Browser Networking(O’Reilly)](https://hpbn.co/webrtc/)
- [WebRTC in the real world: STUN, TURN and signaling - HTML5 Rocks](https://www.html5rocks.com/ko/tutorials/webrtc/infrastructure/)
- [What is a STUN Server and how does it work?](https://www.3cx.com/pbx/what-is-a-stun-server/)
- [Interactive Connectivity Establishment (ICE): A Protocol for Network Address Translator (NAT) Traversal](https://tools.ietf.org/id/draft-ietf-ice-rfc5245bis-14.html)
- [What is WebRTC Signaling?](https://www.onsip.com/voip-resources/voip-fundamentals/webrtc-signaling)
- [https://indigoo.com/petersblog/?p=215](https://indigoo.com/petersblog/?p=215)
- [– Bryan Ford’s Home Page](https://bford.info/pub/net/p2pnat/)
- [NAT를 넘어서 가자](https://snack.planetarium.dev/kor/2019/04/nat_traversal_1/)
- [예제로 보는 TURN](https://snack.planetarium.dev/kor/2019/06/nat_traversal_2/)
- [안드로이드 WebRTC 시작하기 -3](https://juyoung-1008.tistory.com/27)
