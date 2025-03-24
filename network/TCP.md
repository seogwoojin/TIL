# TCP와 Socket

## IP

IP는 순서와 무결성을 보장하지 않는다. 단순히 서버를 찾을 때 사용하는 단서 

결과적으로 파일 전송이나 잘라서 보낼 때, 순서가 뒤죽박죽이 될 수도 있음

## TCP

TCP는 그 위의 레이어로서, 이 단점을 해결해준다. (그래서 너무 균형이 좋아서 TCP/IP 4계층도 생긴듯 하다)

![image](https://github.com/user-attachments/assets/d2ad11db-27a8-43a7-acf3-d434364d1f5b)

## Socket

상위 어플리케이션 레이어와 하위의 시스템 영역을 이어주는 장치가 Socket이다. 

Socket은 어플리케이션이 커널 영역의 요청을 날릴 때 사용하는 커널을 추상화한 인터페이스이다. 


<img width="598" alt="image" src="https://github.com/user-attachments/assets/a7e381c6-5276-4182-8900-06351875ff4c" />

```
serverSocket.accept();
```

내부적으로 블로킹 상태로 들어감. 새로운 tcp 3way shaking이 끝나면, 이 serverSocket을 깨우면 바인딩된 소켓이 이를 읽고, 

커널 소켓과 매핑되는 유저 소켓을 반환한다. 

(쓰레드와 유사한 흐름) => 시스템 객체와 자바 객체의 매핑

<br>

새로운 연결 요청 도착:
클라이언트가 연결 요청(SYN 패킷)을 보내고 3-way 핸드셰이크가 완료되면,
OS는 서버 소켓의 연결 대기열(백로그)에 그 연결 정보를 넣습니다.
이때, 블로킹 상태에 있던 accept()는 해당 이벤트를 감지하고 새로운 소켓을 반환하면서 깨어납니다.

<img width="478" alt="image" src="https://github.com/user-attachments/assets/b3f3bd14-61d5-4ac2-b410-93a671d12e7f" />

정리하면, Tcp 3-way 셰이킹을 통한 연결은 순전히 Tcp Layer에서 일어나는 일

연결이 성립되어 백로그에 이 소켓 연결이 들어가면, 커널이 서버(톰캣)의 블락된 메인 쓰레드를 깨움

## 병목지점이 어디가 될까?

1. 서블릿 수가 부족함 -> 애초에 Default도 200개로 제한돼있음, 요청이 몰린다면 이곳에서 request time out을 반환할 것 같다.

2. Tcp Socket을 더이상 생성할 수 없음? 하드웨어 자원이 부족하거나 제한을 걸어뒀다면, 한 프로세스의 최대 소켓 생성에는 제한이 있지 않을까 싶다.

거기에 keep-alive로 인해 클라이언트와 연결중인 소켓이 늘어나면 문제가 생기지 않을까.. 싶지만 tcp socket이 하드웨어 리소스 소모를 매우 적게 할 것 같아
보통의 경우 1번을 신경쓰는 게 맞는 듯 하다.








