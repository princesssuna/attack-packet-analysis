# ⚖️ Network layer
네트워크 계층은 데이터를 전송할 수 있는 여러 경로 중 가장 안전하고 빠른 경로를 찾아주는 역할인 라우팅을 수행하며, 데이터를 다른 네트워크로 전달하여 인터넷을 가능하게 만드는 계층이다.

## 💡 IP Protocol
네트워크 계층에서 운영되는 IP 프로토콜에는 목적지를 알려주는 IP 주소가 존재한다. 이 IP 주소는 지역을 대표하는 ```네트워크```와 지역별 사용자 PC를 의미하는 ```호스트```로 구분된다. 

IP 주소를 a.b.c.d 라고 가정하였을 때,

클래스|네트워크|호스트|범위
:---:|:---:|:---:|:---:
A|a|b.c.d|0.0.0.0 ~ 127.255.255.255
B|a.b|c.d|128.0.0.0 ~ 192.255.255.255
C|a.b.c|d|192.0.0.0 ~ 223.255.255.255
D|멀티캐스트용|-|224.0.0.0 ~ 239.255.255.255
E|실험용|-|240.0.0.0 ~ 255.255.255.255

일반적으로 **C 클래스**를 많이 쓴다. 

사설 IP 대역은 내부 네트워크에서만 사용하는데, 내부 IP 노출을 차단하거나 부족한 공인 IP를 대신하기 위해 사용한다.

클래스|범위
:---:|:---:
A|10.0.0.1 ~ 10.255.255.254
B|172.16.0.1 ~ 172.31.255.254
C|192.168.0.1 ~ 192.168.255.254

### 💡 IP Header

![ip header](https://user-images.githubusercontent.com/66156026/159121506-28c286e0-d9b3-4192-9a04-3b4f446b7977.gif)

- 🧸 **Version**

IP 버전이 저장되어 있다. 일반적으로 버전 4가 사용된다. 
Wireshark의 Version창을 눌러보면 45의 데이터를 확인할 수 있다. Header Length(IHL)까지 포함된 데이터로 **4는 버전 정보, 5(0101)는 헤더 길이**이다.

![스크린샷 2022-03-19 오후 9 48 05](https://user-images.githubusercontent.com/66156026/159121781-9d9e60c0-eee2-4812-b2fb-029f2fc379ec.png)

- 🧸 **IHL**

Internet Header Length의 약자이며 헤더 길이 정보이다. 마지막 필드인 Options 값에 따라 길이가 가변적이지만, 단위는 32비트(4바이트) 단위이며 **최소 5에서 최대 15**까지 될 수 있다.

구분|크기(bytes)|설명
:---:|:---:|:---:
최소 5일 경우(0101)|20 bytes|(32/8) * 5 = 20
최대 15일 경우(1111)|60 bytes|(32/8) * 15 = 60

기본 크기 20바이트를 제외하고 40바이트가 더 늘어날 수 있다. 

- 🧸 **Type of service**

서비스 종류 및 혼잡 알림을 나타내며, **DSCP 필드(6비트) + ECN 필드(2비트)** 로 구성되어 있다.

![스크린샷 2022-03-19 오후 9 56 33](https://user-images.githubusercontent.com/66156026/159121983-8f23155f-2a46-43fc-b92b-8e517a630dbd.png)

1) **DSCP(Differentiated Service Code Point)**
: 요구되는 서비스의 우선 순위에 대한 유형을 나타내며, IP 데이터그램이 라우터에서 어떻게 처리되어야 하는지 정의되고 있다. CS0 ~ CS7의 클래스로 구분한다. CS0으로 갈 수록 우선 순위가 낮고 CS7로 갈 수록 우선 순위가 높다.
2) **ECN(Explicit Congestion Notification)**
: 혼잡을 알리려고 사용하며, 라우터가 패킷을 즉각 폐기하지 않고 최종 노드에 혼잡을 알리는 용도이다.

구분|내용
:---:|:---:
00|패킷이 ECN 기능을 사용하지 않는다.
01 or 10|발신 측에서 종단점이 ECN 기능을 수용함을 나타낸다.
11|라우터 혼잡이 발생했음을 알리고자하는 표식이다.

- 🧸 **Total length**

헤더와 데이터의 길이를 합한 값이며 최대 ```65,535바이트```까지 사용할 수 있다.

![스크린샷 2022-03-19 오후 10 01 25](https://user-images.githubusercontent.com/66156026/159122107-84670066-5dac-45f0-bf14-223761a81e4c.png)

52의 16진수인 34가 설정되어 있다.

- 🧸 **Identification**

네트워크 기기가 전송할 수 있는 최대 전송 단위를 **MTU(Maximum Transfer Unit)** 라고 한다.
일반적으로 이더넷 환경을 사용하기 때문에 ```1500바이트```로 통용되고 있다.

만약 MTU 이상의 크기의 데이터가 전송된다면, MTU 크기에 맞추어 패킷이 분할되는데 이를 ```단편화(Fragmentation)```라고 한다.
실제 단편화가 발생하면, 데이터의 크기에 따라 수많은 패킷으로 단편화되는데 어떤 패킷에 속한 단편화 패킷인지 구분하기 위해 **고유 번호**가 할당되며 **같은 패킷에서 분할된 패킷들은 같은 ID 값**을 가진다.

**단편화되기 전의 패킷**은 각각 1개의 패킷별로 고유한 ID를 갖게 된다.
**단편화된 패킷**은 이더넷 기준으로 MTU 크기인 1500바이트 이상의 패킷은 각각 단편화되어 전송되는데, 각각의 고유한 번호를 가지고 같은 패킷의 조각에는 같은 ID 값이 부여된다.

- 🧸 **Flags**

단편화된 추가 패킷이 있다는 것을 알려주며, 수신 측에서는 해당 정보를 바탕으로 원래의 패킷으로 재조합한다.

![스크린샷 2022-03-19 오후 10 12 32](https://user-images.githubusercontent.com/66156026/159122543-c663c841-0c91-4891-878e-db131000c2cb.png)

![스크린샷 2022-03-19 오후 10 11 44](https://user-images.githubusercontent.com/66156026/159122547-d5c4e10e-62ed-4b57-b733-58a054c5cf42.png)

구분|내용
:---:|:---:
0비트|예약 필드로 무조건 ```0```으로 설정되어야 한다.
1비트|DF(Don't fragment)에 ```1```이 설정된 경우, 분할된 패킷이 없음을 의미한다.
2비트|MF(More fragments)에 ```1```이 설정된 경우, 분할된 패킷이 더 있을 경우이다. <br/> MF(More fragments)를 비롯한 모든 값에 ```0```이 설정된 경우, 분할된 패킷이 더이상 존재하지 않음으로 의미한다.

- 🧸 **Fragment offset**

분할된 패킷을 수식 측에서 **재배열할 때 패킷들의 순서를 파악**하는 데 사용한다. 
예를 들어 3개의 패킷으로 분할된 경우 다음 순서에 따라 offset이 설정되며, offset은 바로 전까지 보낸 데이터의 크기를 나타낸다.

첫 번째 패킷 : 항상 0으로 설정

두 번째 패킷 : 첫 번째로 보낸 데이터의 크기로 설정

세 번째 패킷 : 두 번째까지 보낸 전체 데이터의 크기로 설정

- 🧸 **Time to live**

패킷 수명을 제한하기 위해 데이터그램이 통과하는 **최대 홉(Hop) 수**를 지정할 수 있으며, 패킷 전달 과정에서 라우터와 같은 전송장비를 통과할 때마다 **TTL(Time to live) 값은 감소**한다.
만약 0이 되면 라우터에서 폐기하여 불필요한 패킷이 네트워크에 방치되는 것을 방지한다.

![스크린샷 2022-03-19 오후 10 24 38](https://user-images.githubusercontent.com/66156026/159122930-2901568f-ef3c-4833-88ff-3f76742d92cc.png)

OS 종류와 버전에 따라 TTL 값은 제각기 다르다.
본인이 사용하는 **MacOS/MacTCP X (10.5.6)** 버전은 ```TCP TTL 64, UDP TTL 64```이다. (Windows 10과 동일하다.)

- 🧸 **Protocol**

IP 헤더에 따라올 상위 프로토콜을 지정하는 것으로, TCP와 UDP, ICMP 등의 프로토콜 종류를 확인할 수 있다.

![스크린샷 2022-03-19 오후 10 29 05](https://user-images.githubusercontent.com/66156026/159123087-5613b42e-3a74-43e6-8734-e3d5e6f946dd.png)

**TCP**는 ```6```, **UDP**는 ```17```, **ICMP**는 ```1```, **GRE**는 ```47``` 이다.

- 🧸 **Header checksum**

**헤더의 오류를 검증**하기 위해 사용한다.

![스크린샷 2022-03-19 오후 10 34 38](https://user-images.githubusercontent.com/66156026/159123243-2cc0e2d4-5d8d-4e83-a927-6965bac1b2ce.png)

오랜만에 일년 전, 과제로 나왔던 checksum 계산 자료를 꺼내보았다.

version 값부터 목적지 IP 값까지 데이터를 2바이트 단위로 나눠서 더해주고 올림수가 나온다면 하위 바이트에 더해준다. 이 더한 값에 1의 보수를 취하면 checksum 값이 나온다.
보수화된 값을 16진수로 표현하여 올바른 checksum 값인지 확인한다. 이런 방식으로 수신 측에서 에러 발생 여부를 확인할 수 있다.

- 🧸 **Source address**

송신자(출발지)의 IP 주솟값이 설정된다.

- 🧸 **Destination address**

수신자(목적지)의 IP 주솟값이 설정된다.

- 🧸 **Option + Padding**

새로운 실험 혹은 헤더 정보에 추가 정보를 표시하기 위해 설계되었으며, IP 헤더의 필수 항목은 아니다. 유용한 제어/시험/디버깅을 할 수 있지만, 통신 자체에는 관여하지 않으며 현재 옵션 필드는 거의 사용되지 않는다.
앞에서 말한 IHL이 5를 초과하였을 때 이 옵션 필드가 사용된 경우이다.

