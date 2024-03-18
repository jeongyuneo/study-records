# OSI 7 Layers

## 1. OSI 7 Layers

OSI 7계층은 네트워크 통신 시 작동하는 기능을 7계층으로 나눈 구조로, 범용적인 네트워크 구조다.

OSI 7계층의 등장으로 네트워크 통신 흐름을 한눈에 알아보기 쉬워지고, 특정 부분에 문제가 생기면 다른 장비나 소프트웨어를 건드리지 않고 이상이 생긴 곳만 고칠 수 있게 되었다.

또다른 네트워크 계층 구조로 인터넷에 특화된 TCP/IP 4계층이 있다.

![](https://velog.velcdn.com/images/minide/post/5c37865d-7ec7-4739-8117-b9e5a16b10b3/image.png)

- 각 레이어에 맞게 프로토콜이 세분화되어 구현
- 각 레이어의 프로토콜은 하위 레이어의 프로토콜이 제공하는 기능을 사용해 동작

### 1.1. 1st - Physical Layer

비트 단위로 데이터를 전송하는 계층

### 1.2. 2nd - Data Link Layer

직접 연결된 노트 간의 통신을 담당하는 계층

- MAC 주소 기반 통신
- **흐름제어**: 수신자의 수신 데이터 전송률을 고려해 데이터를 전송하도록 제어
- **오류제어**: 손상 또는 손실된 프레임을 발견 및 재전송(트레일러를 통해 이루어짐)
- **접근제어**: 주어진 어느 한 순간에 하나의 장치만 동작하도록 제어

ex. ARP, RARP

### 1.3. 3rd - Network Layer

호스트 간의 통신을 담당하는 계층

- 목적지 호스트로 데이터 전송
- 네트워크 간 최적의 경로 결정

ex. IP

### 1.4. 4th - Transport Layer

애플리케이션 간의 통신을 담당하는 계층

- 목적지 애플리케이션으로 데이터 전송

ex. TCP(안정적이고 신뢰성있는 데이터 전송 보장), UDP(필수 기능만 제공)

### 1.5. 5th - Session Layer

애플리케이션 간의 통신에서 세션을 관리하는 계층

- TCP/IP 세션 생성 및 제거

ex. RPC(Remote Procedure Call) 등

### 1.6. 6th - Presentation Layer

애플리케이션 간의 통신에서 메시지 포맷을 관리하는 계층

ex. 데이터 변환(인코딩 ↔ 디코딩), 암호화(암호화 ↔ 복호화), 압축(압축 ↔ 압축 해제) 등

### 1.7. 7th - Application Layer

애플리케이션 목적에 맞는 통신 방법을 제공하는 계층

ex. HTTP, DNS, SMTP, FTP 등

## 2. OSI 7 Layers Process

![](https://velog.velcdn.com/images/minide/post/9d464753-bcc4-4506-a61f-7b531ca02dbf/image.png)

- 캡슐화(Encapsulation): 데이터를 네트워크로 전송하기 전에 필요한 정보를 헤더 또는 트레일러로 데이터를 포장하는 것
- 역캡슐화(Decapsulation): 데이터를 네트워크로 전송하기 전에 필요한 정보가 담긴 헤더 또는 트레일러를 데이터에서 떼어내는 것

(트레일러는 데이터링크 계층에서만 사용되며, 데이터 전송 후 에러가 없었는지 확인하기 위해 사용하는 용도임)

## References

[[입문용] 프로토콜과 OSI 7 layer 설명! 네트워크의 기능들이 어떻게 구조화 돼서 동작하는지를 설명합니다! 👍](https://www.youtube.com/watch?v=6l7xP7AnB64)

[OSI 7계층 모델 - 핵심 총정리](https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-OSI-7%EA%B3%84%EC%B8%B5-%EC%A0%95%EB%A6%AC)
