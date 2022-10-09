블로그: https://velog.io/@destiny1616/TCP-3-Way-%EB%B0%8F-4-Way-Handshake

TCP 3-Way Handshake 프로세스

## TCP

TCP(Transmission Control Protocol)는 OSI model의 transport layer에서 사용되는 연결지향적인 프로토콜이다. 좀 더 자세히 살펴보자.

#### 1. TCP는 연결지향이다.

이는 송신자와 수신자가 반드시 서로 연결되어서 데이터를 주고 받는 것에 동의해야 한다는 의미이다. 이러한 목적으로 TCP는 3-Way Handshake라는 기술을 사용한다.

### 2. TCP는 신뢰할 수 있는 프로토콜이다.
에러체크 기능이나 순서가 보장된 데이터 스트림을 제공한다. 전송된 패킷들은 프로토콜 스택의 다른 계층들에서 손실될 수도 있다. TCP는 이러한 문제를 탐지해낼 수 있다. 이러한 경우 TCP는 손실된 패킷을 재전송한다. 또한 순서가 뒤바뀐 패킷을 재배열할 수도 있다. 만약 패킷을 정상적으로 처리할 수 없을 경우에는 송신자에게 해당 문제를 알린다.

### 3. TCP는 유니캐스트이며 양방향이다. 

송수신자의 1대1 handshake의 결과이다. TCP는 여러 호스트를 처리할 수 없다. 


TCP 프로토콜의 시작은 바로 3 way handshake이다. 해당 작업을 통해 클라이언트와 서버가 연결이 되고 데이터 송수신 과정이 시작되는 것이다.
이러한 송수신 과정에서 세그먼트라 불리우는 transport layer의 Protocol Data Unit(PDU)을 사용한다. 
이 세그먼트 헤더의 제어 비트로 TCP 연결을 이뤄내고 관리할 수 있다.

## 제어 비트(Control Bits)

세그먼트 헤더에는 소스 포트, 목적지 포트, 패킷 시퀀스 번호, 체크섬, 플래그 등 중요한 정보를 담고 있다. 

https://www.baeldung.com/wp-content/uploads/sites/4/2021/02/tcp-header-segment.svg

연결을 위해 SYN flag와 ACK flag가 필요하다. 이 플래그들은 헤더에 있는 비트이며 값 1로 on을, 값 0으로 off를 나타낸다. 이 플래그는 TCP 클라이언트와 서버가 그들이 수신한 패킷으로 무엇을 할지나 어떻게 응답할지를 결정할 때 사용된다. 
연결을 하기 위해 어떻게 SYN flag를 사용하는지 살펴보자.

## 3-Way Handshake

3-Way Handshake는 이름 그대로 연결을 하기 위해 3단계의 핸드쉐이크를 해야함을 뜻한다. 이 각 단계에서는 특정 플래그가 필요하다.

https://media.geeksforgeeks.org/wp-content/uploads/TCP-connection-1.png

### 1. SYN

먼저 클라이언트는 헤더에 sequence number와 SYN 플래그 비트만 세팅한 패킷을 보낸다. 이 초기 패킷을 통해 클라이언트는 클라이언트로부터의(originating from the client) 요청 패킷에 대한 첫 번째 sequence number를 설정할 수 있다. 이를 클라이언트의 동기화 단계라 한다.

### 2. SYN/ACK

서버는 SYN 패킷에 SYN/ACK으로 응답한다. 서버에서는 SYN 플래그 비트와 ACK 플래그 비트를 세팅한다. 그리고 이 패킷에서는 클라이언트가 보낸 sequence number를 승인하여 확인한다(acknowledging). 이 과정은 보통은 클라이언트에서 보낸 sequence number에 1을 더하는 식으로 세팅된다.  
또한 서버로부터의(originating from the server) 응답 패킷에 대한 첫 번째 시퀀스 넘버가 무엇인지를 세팅하기 위해 서버 역시 클라이언트에 SYN과 sequence number를 보내게 된다. 이 응답패킷은 서버의 동기화 단계이다.

### 3. ACK

마지막으로 클라이언트는 SYN/ACK 패킷에 ACK 패킷으로 응답한다. 위 그림에서 보다시피 서버의 sequence number에 1을 더하는 식이다.

이 3가지 단계를 거쳐 3-Way Handshake가 완료되고 sequence number가 동기화 되며 연결이 시작되어 데이터를 송수신할 수 있게 된다. 

## 4-Way Handshake

연결할 때와 마찬가지로 해당 연결을 종료할 때에도 Handshake가 필요하다. 이때에는 4-Way Handshake를 거치게 된다.
연결을 할 때에는 SYN 비트를 사용했으나, 종료할 때에는 FIN이라는 플래그 비트를 이용한다. 

https://media.geeksforgeeks.org/wp-content/uploads/CN.png

### 1. FIN From Client

클라이언트 측에서 연결을 닫기로 했다고 가정하자(서버측에서 닫을 수도 있다). 클라이언트는 FIN 비트를 1로 세팅한 세그먼트를 서버로 보내고 FIN_WAIT_1 상태로 들어가게 된다. FIN_WAIT_1 상태에 있는 동안 클라이언트는 서버로부터의 ACK 응답을 기다리게 된다.

### 2-1. ACK From Server

서버가 클라이언트로부터 FIN 비트가 세팅된 세그먼트를 수신하면, 서버는 즉시 ACK 비트를 세팅한 세그먼트를 클라이언트로 보낸다. 

### 2-2. Client waiting

FIN_WAIT_1 상태에 있는 동안 클라이언트는 서버로부터의 ACK 응답을 기다리고 있다 했다. 이 세그먼트를 수신하게 되면 클라이언트는 FIN_WAIT_2 상태로 들어가게 된다. FIN_WAIT_2 상태에 있는 동안 클라이언트는 서버가 FIN 비트가 1로 세팅된 세그먼트를 보낼때까지 대기하게 된다.

### 3. FIN from Server

서버는 ACK 세그먼트를 보내고 난 후 일정시간이 지나면 FIN 비트를 세팅한 세그먼트를 클라이언트로 보내게 된다.

### 4. ACK from Client

클라이언트가 서버로부터 FIN 비트 세그먼트를 수신하면, 클라이언트는 ACK 비트 세그먼트를 서버에 보내고 TIME_WAIT 상태로 들어간다. TIME_WAIT 상태는 만약 ACK가 손실된 케이스에 클라이언트가 최종 ACK을 다시 보낼 수 있게 한다. TIME_WAIT는 구현에 따라 다르지만 일반적으로 30초 혹은 1분, 2분 이다. 일정시간을 대기한 후 연결은 공식적으로 닫히고 클라이언트 측의 모든 리소스(포트 번호 및 버퍼데이터 등)가 해제된다.


## 왜 TCP 연결을 종료하기 위해서는 4단계의 Handshake가 필요한가?

왜 연결 종료시에는 ACK/FIN을 한 번에 보낼 수 없는 것일까? 

사실 4-Way Handshake는 자세히 보면 한 쌍의 2-Way Handshake로 작동한다. 첫 번째 단계에서는 클라이언트가 FIN 플래그를 서버에 보낼 때 반환값으로 서버가 ACK 플래그가 담긴 세그먼트를 보낸다. 

```
Client ------FIN-----> Server
Client <-----ACK------ Server
```

이 시점에서 클라이언트는 추가적인 대기 상태로 빠지며, 서버로부터의 FIN 플래그를 기다린다. 이 단계에서 클라이언트 측의 연결은 종료가 가능하다. 그리고 이 추가적인 대기 상태는 FIN_WAIT_2 이다.

이 상태에서 클라이언트 측은 더 이상 추가적인 데이터를 보낼 수는 없으나 서버로부터 데이터 수신은 가능한 상태이다.

그리고 서버가 데이터 전송을 끝내게 되면, 서버는 이제 종료 요청으로 FIN 플래그를 클라이언트로 보내게 된다. 그리고 클라이언트는 연결 종료에 대한 확인으로 ACK 플래그를 서버 쪽으로 보내게 된다.

```
Client <-----FIN------ Server
Client ------ACK-----> Server
```

정리하자면, 스텝 2와 3은 서로 다른 상태에 속하기 때문에 하나의 패키지로 묶어서 보낼 수가 없다.
클라이언트에 의해 서버로 보내진 첫 번째 FIN 플래그는 종료에 대한 요청이다. 그리고 클라이언트가 서버로부터 수신한 첫 번째 ACK는 그저 FIN1에 대한 응답이다. 이제 클라이언트로부터의 연결만 끊겼을 뿐 서버는 여전히 작동 중이다. 이는 전송해야할 데이터가 아직 남아있을 수 있다는 것을 의미한다. 이 상태에서는 갑자기 연결이 끊어질 수는 없다. 모든 데이터 전송이 끝난 후에, 서버는 추가적인 2 스텝, 즉 FIN-ACK 과정을 수행하여 최종적으로 연결을 끊는다.