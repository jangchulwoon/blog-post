---
layout: post
title:  HighPerformance-Web
tags: HighPerformance-Web WEB
categories: HighPerformance-Web
---   

   
#### Rule.0 개요    

웹 브라우저에서 한 페이지가 로딩될 때를 100으로 본다면 실제 HTML이 로딩되는건 10-20 밖에 안됨. 나머지 80-90은 `구성요소`를 다운받는데 사용된다. 즉, Cache가 비어있다고 가정한다면 WEB은 HTML 보다 그 외 요소들을 받기위해 더 많은 시간을 쏟는다.  

> Front에서 대부분의 성능은 구성요소에 의해 결정된다.  

#### Browser Cache   

특별한 헤더를 사용하지 않는다면, 브라우저는 Cache된 데이터가 유효한지 알아보기 위해, Optional Get Request (조건부 Get 요청)을 날린다. 이 때, If-Modified-Since 라는 헤더에 최종 수정날짜를 입력해 Server에게 요청을 보내고 서버는 변경사항이 없다면 304 - Not Modified  라는 응답을 주게된다.    
> 여기서의 문제점은 Cache된 Data가 유효한지 알기 위해, 별도의 요청을 보낸다는 점이다.    
( 뒷 장에 나오겠지만 성능을 줄이는 방법중 하나는 HTTP 요청을 줄이는 것이다. )   
이와같은 문제점을 해결하기 위해 등장한 Header 가 Expires 라는 헤더로 만료기한이 지나지 않았으면 Browser는 HTTP요청을 피한다.
해당 방법은 `Rule.3`에서 자세히 설명한다.    

#### Keep Alive     

HTTP는 비연결형 프로토콜로 응답 후, Connection을 끊어버린다. 웹페이지를 보여주기 위해 필요한 파일들(.js .css .png)은 절대 적은 수가 아니며, 그때마다 Connection을 맺는건 비 효율적이다. 

> 주로 한 서버(ex IMG Server)에 여러 요청을 보내기 때문에 소수의 Connection으로 다수의 요청을 받아 처리하는게 효율적이다.

때문에 HTTP 1.1부터 Keep-Alive 라는 헤더가 나왔고, 이를 사용하면 일정 시간동안 Connection을 유지한체 트랜잭션을 처리할 수 있다.

	Keep-Alive: timeout=5, max=100
	Connection: Keep-Alive

별도의 설정을 안해주면 [15초](https://devcentral.f5.com/questions/tcp-keepalive-v-http-keepalive-55713)라고 한다.

So in effect, http keep-alive timeout overrides the TCP one. If conneciton is closed for any reason, then client must initiate a new connection to send a new request.


#### TCP 와 HTTP의 Keep alive 차이 

Apache는 아니지만, TCP Keep Alive 와 HTTP Keep Alive의 차이점이 나와있는 글 
[tcp keepalive 와 nginx keep alive](https://brunch.co.kr/@alden/9)  

들어가기전에 TCP 와 HTTP 각 Layer에서 어떻게 Connection 관리를 하는지 검토해본다.    

#### TCP Connection 

TCP의 KeepAlive는 관련 Parameter 중 연관된 3개의 값을 통해 이해할 수 있다.
	
	keepalive_time # keepAlive의 유지시간 default 7200

	keepalive_probes # keepAlive가 끊어졌다고 판단하고 세션을 정리하는 동안 보낼 Ping의 수 default 9

	keepalive_intvl # 첫번째 health_check 후 ping-pong 패킷을 보내는데 걸리는 패킷 사이 주기. default 75

> keepalive_time는 이해하기 쉽지만 .. 밑에 껀 ..?  

두 노드간 TCP 세션이 있다고 가정하고, 아무런 통신도 하지않는 Idle 상태가 된지 7200초 후, 살아있는지를 체크하는 ping-pong 패킷을 보낸다. (아무런 의미없는 ACK를 보냄) 그 후, 상대방으로부터 ACK를 받지 못하면 최대 9번까지 75초 주기로 ping-pong을 보내고 세션을 정리한다.    

이와 같은 과정들을 OS Level 에서 관리한다. 

> setsockopt를 이용해 SO_KEEPALIVE 옵션을 설정한다. 

#### HTTP Connection 

nginx 의 keepAlive 는 조금 다른데, OS level이 아닌 Application Level의 Keep Alive이다.  

> 사실 아직 이 문장을 이해 할 수가 없다.    

해당 포스팅의 말을 빌리자면, Tcp keep alive를 사용하지 않으며, tcp 처럼 ping-pong을 통해 상대방이 살아있는지 확인하지도 않는다. 즉 사전에 설정된 keepAlive 시간동안 세션을 유지하는 방식이다.    


#### 차이점   

> 이 부분은 이해한 것을 토대로 작성한 것으로, 실제와 다를 수 있다.    

이해한 차이점을 설정하자면, TCP Keep Alive의 경우 connection의 유지/관리를 커널이 직접하게 된다. 즉, TCP 스택에서 직접 해당 소켓이 살아있는지를 확인하는 작업을 진행한다.   

HTTP Keep Alive는 tcp keepalive를 사용하진 않으며, tcp-keep alive 처럼 ping-pong을 통해 상태방이 살아있는지를 확인하지도 않는다. 즉, keepAlive의 설정된 시간이 지나면 client와 연결을 능동적으로 끊는다. 

> 해당 글에서는 TCP keepAlive와 nginx keepAlive는 연관성이 없다.   
> TCP keepAlive 설정을 건든다고, nginx의 설정에 영향이 가지 않으며 Apache도 마찬가지.  


눈여겨 봐야할 부분은, SO_KEEPALIVE를 사용하는지, 아니면 Application 자체적으로 keepAlive로직을 구현한것인지 정확히 알고 있어야한다. 

> TCP에 의해 관리되는가 Application에 의해 관리 되는가를 명확히 해야함 


#### TCP Keep Alive는 언제 필요하지? 

> 이 의문점은 사실 TCP만으로 돌고있는 시스템(?)에 대한 지식이 없어서 생긴 것으로, 대부분 HTTP KeepAlive만 사용했기 때문 ..

해당 글에서는 이를 LB(Load Balancing)와 연관시켜 설명하는데, 링크를 타고 들어가서 보면 좋을 것 같다. 

(알 수 없는 용어가 많이 나와서 ... )

간단하게 설명하자면, Client 와 LB , 그리고 LB에 의해 관리되어지는 A , B Server가 있다고 가정하자.

1. Client는 LB와 Connection Pool을 맺으며, LB는 상황에 따라 요청을 A 혹은 B서버에게 넘긴다. 
2. 이 상황에서 LB에 Session Table이 붙어 있고 expire 시간이 10분 이라고 했을 때 만약 사용자가 11분 후, 요청을 날리면 세션은 만료되고 LB는 새로운 요청을 받아 A 혹은 B서버에게 날린다. 
3. 만약 초기 A서버에 연결이 됐고, 세션만료후 B서버에 연결이 될 경우, 문제가 발생하는데 문제는 다음과 같다. 


	1. A서버는 클라이언트로 부터 FIN 패킷조차 받을수 없기에 계속 기다리다가 장애 발생 
	2. Client는 영문도 모른채 RST 패킷을 받고 타임아웃 후, 다시 SYN를 맺고 B와 연결 

(세션이 없는 요청의 경우 RST 를 보낸후 SYN을 보낸다고 한다.)

이런 경우, TCP KeepAlive를 사용하면 A는 지속적으로 ping pong을 받기 때문에 세션 테이블에서 지워지지 않고, 계속 정보가 유지된다 고 한다.

> 여기서 의문점은 LB의 세션은 내가 아는 세션과는 조금 다른거 같은데?  

#### Body 압축       

HTTP 요청 수 뿐만아니라 Body의 용량또한 성능에 영향을 미친다. 
이를 보완하기 위해 Browser와 Server간의 협의(?)를 통해 압축된 Body로 응답을 받을 수 있다. 자세한 부분은 Rule.4 에서 설명한다. 


책 초장에 나와있는 챕터로, 책을 읽기 전 필요한 HTTP 개념에 대해 서술한 부분이다.
알고있었지만 명쾌하지 않았던 부분들이 있어서 개념을 정리하는데 좋은 시간이었다. 
