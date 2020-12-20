---
title: WebRTC란? PeerJS 코드와 비교해보기
layout: post
date: '2020-12-19 21:00:00'
author: 진혀쿠
tags: 자바스크립트 JavaScript WebRTC 음성채팅 화상채팅
cover: "/assets/instacode.png"
categories: JS
---

5주 자유 프로젝트를 위해 사용했던 WebRTC에 대해 좀 더 알아보고자 글을 정리하기로 했다. 우리는 PeerJS라는 라이브러리를 활용하여 조금 쉽게 코드를 짰는데 실제 WebRTC는 어떻게 이루어져 있는지 궁금하여 학습을 해보았다.

## WebRTC란?

WebRTC 공식문서와 MDN의 설명을 빌려 다음과 같이 정리해보았다.

>WebRTC는 웹 표준으로 구현되었으며 P2P 방식을 활용하여 웹 애플리케이션과 사이트가 중간자 없이 브라우저 간에 오디오나 영상 미디어를 받아와 스트림할 수 있게 해주며 데이터 교환 역시 가능하게 해준다. 
WebRTC는 플러그인이나 다른 소프트웨어의 설치가 필요 없으며 대부분의 브라우저에서 JavaScript API로 지원되고 있다.

## WebRTC의 3가지 주요 API

WebRTC에서는 여러 API를 제공하고 있으나 그 중에서도 핵심이 되는 3가지 API가 있다.

### getUserMedia
화상 통화를 하기 위해서는 가장 먼저 비디오가 준비되어야 한다. 이를 위해 존재하는 API이다. getUserMedia는 유저의 일치하는 장치에 접근하여 MediaStream을 Promise로 return 시킨다. constraint를 설정하여 조건을 지정할 수 있다.

### RTCPeerConnection
각 피어의 연결을 담당하는 API이다. 이 객체는 RTCConfiguration을 인자로 전달 받는데 인자에는 피어 연결이 설정되는 방법과 ICE 서버에 대한 정보를 포함시켜야 한다. RTCPeerConnection 이 생성되면 호출 피어인지 수신 피어인지에 따라 SDP 제안 또는 응답을 생성해야한다. 또한 SDP 제안 또는 응답이 생성되면 다른 채널을 통해 원격 피어로 전송되어야한다.

### RTCDataChannel
RTCPeerConnection을 통해 데이터를 전송하기 위한 API이다. RTCPeerConnection 객체에서 createDataChannel()을 호출하여 얻을 수 있다. 원격 피어는 RTCPeerConnection 개체에서 datachannel 이벤트를 수신하여 데이터 채널을 수신할 수 있다.

## 모르는 단어 정리
### ICE 서버란?
ICE란 Iteractive Connectively Establishment의 약어이다. 두 단말이 서로 통신할 수 있는 최적의 경로를 찾을 수 있도록 도와주는 프레임워크이다. ICE는 STUN(Session Traversal Utilities for NAT)와 TURN(Traversal Using Relay NAT)을 활용하는 프레임워크로 SDP 제안 및 수락 모델에 적용할 수 있다.

### STUN과 TURN 간단 정리
STUN은 단말이 자신의 공인 IP 주소와 포트를 확인하는 과정에 대한 프로토콜이고, TURN은 단말이 패킷을 릴레이 시켜 줄 서버를 확인하는 과정에 대한 프로토콜이다. STUN 서버는 사설 주소와 공인 주소를 바인딩하고, TURN 서버는 릴레이 주소를 할당한다. 특히 TURN은 ICE에서 직접 사용한다.

### SDP란?
SDP는 신원을 기반으로 리소스에 대해 액세스를 제어하는 프레임워크로 네트워크 장치, 단말의 상태, 사용자 ID를 체크하여 권한이 있는 사용자 및 디바이스에 대해서만 액세스 권한을 부여하며 인증 받지 못한 단말기에 대해서는 그 어떠한 서비스 연결 정보도 얻지 못하게 된다. 인프라는 인증 및 인가가 되기 전에는 DNS정보나 IP주소를 알 수 없는 '블랙 클라우드(Black Cloud)' 네트워크로 동작이 되면서 해커들이 쉽게 보안을 뚫을 수 없도록 구성이 되어 있다.

## Signaling
RTCPeerConnection이 생성되기 이전에 서로 다른 네트워크에 있는 2개의 디바이스들끼리의 통신을 위해 각 디바이스들의 위치를 발견하는 방법과 미디어 포맷에 대한 협의가 필요하다. 이 과정을 Signaling이라고 부른다. Signaling은 3가지 종류의 정보를 교환한다.

- Session control messages
- Network configuration
- Media capabilities

Signaling을 통한 정보 교환은 P2P streaming이 시작되기 전에 반드시 성공적으로 완료 되어야 한다.

<img src="{{ site.baseurl }}/assets/peerjs/signaling.png" title="peerjs signaling" class="picture">

## 우리가 짠 코드와 비교해보기
그렇다면 위의 3가지 API들이 우리의 코드에는 어떤식으로 활용되고 있을까? 

### getUserMedia
PeerJS에서도 getUserMedia는 그대로 활용하고 있다.

### RTCPeerConnection
PeerJS에서는 RTCPeerConnection을 call이라는 메소드를 활용해 처리하고 있다. 실제 WebRTC에서는 createOffer를 통해 원격 peer에게 호출을 하는데 PeerJS에서는 call을 통해 자신의 stream을 전달하고 상대방의 stream을 받아온다. 위의 설명에서 원격 피어는 호출 받았을 시 응답을 생성해야 한다고 했는데 PeerJS에서도 실제 WebRTC의 API인 createAnswer와 유사하게 answer라는 메소드를 통해 응답을 하고 있다.  
또한 위에서 ICE 서버에 대한 정보를 담고 있어야 한다고 나와있는데 이는 PeerJS 라이브러리가 제공하는 Peer 클래스를 생성할 때 설정할 수 있는 듯 하다. (이 부분은 PeerJS의 코드를 직접 확인하지 않아서 확신할 수 없습니다. 잘못된 정보라면 알려주세요!)

### RTCDataChannel
이 부분은 조금 헷갈리는 부분이다. 내가 본 WebRTC 공식 문서 가이드에서는 RTCDataChannel을 통해 Message를 전송하는 예제를 다루고 있는데 PeerJS를 확인해봤더니 DataConnection과 MediaConnection 두 개의 객체에 대한 설명이 따로 있었다. DataConnection의 설명을 보면 WebRTC의 DataChannel을 감싸고 있는 객체라고 설명이 되어 있다. 또한 각각을 얻는 메소드도 connect과 call로 구분이 되어 있다. 우리가 코드에서 활용한 call에서는 MediaConnection이라는 객체를 return하고 있다. 메시지와 영상/음성 데이터의 전송이 따로 관리되고 있는 건가 라는 생각이 들어서 WebRTC의 dataChannel에 대해 좀 더 검색을 해봤더니 MDN 문서에서 다음과 같이 설명하는 것을 확인할 수 있었다.

>RTCPeerConnection 인터페이스의 createDataChannel() 메소드는 어떤 형식의 데이터든 송신 할 수 있도록 원격 유저와 연결하는 신규 채널을 생성합니다.
이 방법은 이미지, 파일 전송, 문자 채팅, 게임 패킷 업데이트 등과 같은 백채널 컨텐츠에 유용하게 사용 가능합니다.


결론적으로 WebRTC에서는 dataChannel을 통해 모든 종류의 데이터를 주고 받고 있다는 것인데 PeerJS에서는 왜 나누어져 있는지 의문이다.
PeerJS 공식문서의 connect와 call에 대한 설명에서도 큰 차이를 찾을 수가 없었다.

<img src="{{ site.baseurl }}/assets/peerjs/connect.PNG" title="peerjs connect method" class="picture">

<img src="{{ site.baseurl }}/assets/peerjs/call.PNG" title="peerjs call method" class="picture">

코드를 직접 확인할 수 있는 시간이 난다면 확인해보고 내용을 추가할 예정이다.

## 참고 사이트 및 링크들
- <https://webrtc.org/>
- <https://www.html5rocks.com/ko/tutorials/webrtc/basics/>
- MDN
- <https://brunch.co.kr/@linecard/156>
- <https://peerjs.com>
- <https://ko.wikipedia.org/wiki/%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4_%EC%A0%95%EC%9D%98_%EA%B2%BD%EA%B3%84>