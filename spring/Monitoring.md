# `스프링 부트 모니터링`

### **전투에서 실패한 지휘관은 용서할 수 있지만 경계에서 실패하는 지휘관은 용서할 수 없다**

## Actuator

스프링 부트에서 제공해주는 기능으로 JVM 위의 모든 정보와 하드웨어 상태를 파악할 수 있습니다.

또한 헬스 체크를 통해서 ALB의 로드밸런싱을 돕기도 하구요.

(https://toss.tech/article/how-to-work-health-check-in-spring-boot-actuator 참고할만한 글)

### +) Actuator의 보안

이러한 정보는 해커의 먹이가 될 수 있기 때문에, 기본 경로로 사용해선 안됩니다. 

이에 대한 보안 적용을 위해선 다음을 고려할 수 있을 것 같습니다. 

- 인증, 인가 추가
- VPN 내부 접속만 허용 되도록
- 액츄에이터 경로 변경
- 꼭 필요한 정보만 enable 하게 올리기

  <br>

하지만 이러한 액츄에이터 하나로는 이 정보들이 저장되지 않고 사라집니다. 

단편적인 사진 한장만 작성하는 거죠. 이제 이 프레임을 이어붙여 동영상을 만들고 이 흐름을 감지해야 합니다.

각 순간의 데이터는 어떻게 저장하는가?

## 프로메테우스

이에 대한 대답은 프로메테우스라고 할 수 있습니다. 

오픈소스로 된 모니터링 툴로, 스프링 액츄에이터와 연결을 통해 메트릭 데이터를 특정 간격으로 수집할 수 있습니다.

![image](https://github.com/user-attachments/assets/4072a3ff-4c56-4ec9-bf59-5d98c313cd52)

```
implementation 'org.springframework.boot:spring-boot-starter-actuator'
runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
```

추가적인 의존성을 추가해줍니다. 일반적인 Json 메트릭은 프로메테우스가 수집하지 못하기 때문입니다.

<img width="548" alt="image" src="https://github.com/user-attachments/assets/6fda4617-b40e-42a4-942a-10125d33b9ef" />

스프링의 강력한 추상화는 여기서도 위력을 발휘하네요. 
마이크로미터를 통해 어떤 시중의 모니터링 툴에도 적용 가능하게 설계 됐습니다. 

툴이 바뀌더라도 구현체만 바꾸면 끝


<img width="539" alt="image" src="https://github.com/user-attachments/assets/3199c67f-9056-480e-8b21-a7eb56f76088" />

프로메테우스 만으로도 데이터 시각화가 '일부' 가능하긴 하나, 한눈에 알아보기 어려운 경향이 있어 UI를 붙이게 됩니다.

## 그라파나

그라파나 역시 오픈 소스로, 프로메테우스의 PromQL을 사용하여 데이터를 받아와, 이를 그려줍니다

<img width="524" alt="image" src="https://github.com/user-attachments/assets/a954b9c5-4a42-45bc-b53a-95736d8805eb" />

이렇게 얻은 데이터로 보통 무엇을 체크에 어떤 상황에 대비할 수 있을까요?

1. Connection Pool
   
유저가 급증해, Tomcat에 유휴 Pool이 없거나, DB Connection Pool이 계속 차있어 타임아웃이 나는 경우를 체크할 수 있습니다.

이 경우에는 서버 확장이나 DB 확장 또는 풀 증가를 고려할 수 있을 것 같습니다. 

2. CPU 사용량

서버에 계산이 많아진다면, 이후 들어오는 응답을 빠르게 처리하지 못합니다. 이를 CPU 사용량 모니터링을 통해 방지할 수 있습니다.

3. JVM 메모리 사용량 초과

Garbage Collector에 의해 회수되지 않는 메모리 누수가 발생하고 있는 경우
서버에서 새고 있는 포인트를 빠르게 찾아야 합니다.

4. Error 로그 급증

서버에서 Error 로그가 급증하고 있다는 뜻은 긍정적인 신호가 아닙니다.


##  +) 분산 환경에서의 핀포인트

이런 환경에선 네이버에서 개발한 핀포인트를 강의를 통해 추천받았습니다. 관련해서 간략하게 찾아서 적용해볼 예정

https://d2.naver.com/helloworld/1194202






