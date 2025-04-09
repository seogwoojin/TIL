# 이펙티브 자바 - 동시성

자바는 강력한 멀티 쓰레딩이 가능한 언어로, 멀티 코어 환경에 적합한 언어이다.


![image](https://github.com/user-attachments/assets/bd0366aa-cecd-4f9d-81cc-8566e3727ca8)

하지만 강력한 힘에는 책임이 따르는 법. 동시성 문제는 프로그래머가 고려해야할 문제이다.

```java
public class Main {

    public static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread backgroundThread = new Thread(()->{
            int i = 0;
            while (!stopRequested){
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }

}
```

간단하지만, 끝나지 않는 코드. 

jvm이 내부적으로 Hoisting 이라는 최적화 기법을 사용하기에 문제가 생긴다. 

```
if (!stopRequested)
  while (true)
    i++;
```

동기화 설정이 없다면 최적화 후 아래와 같이 변하고 값의 변화를 인지하지 못한다.


```java
public class Main {
    public static synchronized void requestStop(){
        stopRequested  = true;
    }

    public static synchronized boolean stopRequested(){
        return stopRequested;
    }
    public static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread backgroundThread = new Thread(()->{
            int i = 0;
            while (!stopRequested()){
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}

```

수정 코드. 사실 이경우는 더 좋은 방법이 있다

```java
public class Main {

    public static volatile boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {

        Thread backgroundThread = new Thread(()->{
            int i = 0;
            while (!stopRequested){
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }

}
```
volatile 한정자는 동기화 와는 상관 없지만, 항상 최신의 메모리 결과를 읽도록 한다. 즉 최적화를 방지한다.

이런 저런 방법들로 스레드 간의 공유 가변 데이터에 대한 동시성을 제어했다.

하지만 이건 분명하다. **가장 좋은 해결책은 가변 데이터를 공유하지 않는 것이다.**


<br>

## 동기화와 성능, 교착상태

동기화 (synchronized) 내부에서 순진하게 외부 객체를 부르거나 하는 행위를 지양하자

거기에 무엇이 담겨있을 지, 얼마나 걸릴지 모른다. -> 교착상태 발생 가능성이 생기고, 블락킹 시간 증가

**따라서 동기화는 필요할 때만 간결하게 사용하고 빠르게 해제하는 것이 핵심이다.**


<br>

## 성능이 필요하다면 내부 동기화, 그 외에는 외부에서 관리

내부 동기화는 빠르지만 위험하다.

오히려 클래스나 메소드에 대해 스레드 안전하지 않다는 것을 알려주고 클라이언트에서 관리하도록 시키는 것이 좋은 경우가 많다.

<br>

## 추가적으로, [스프링캠프 2023 [Session 6] 구현부터 테스트까지 - 대용량 트래픽 처리 시스템 (이경일) - 대규모 처리 관련 영상](https://www.youtube.com/watch?v=XBXmHCy1EBA&t=1750s)

**Synchronzied** = 화장실에 문을 잠구고 들어가는 것

가장 안전하지만 너무 비쌈. 실질 운영 환경에서는 불가능

<img width="632" alt="image" src="https://github.com/user-attachments/assets/affe4e5b-4919-4b94-bcdf-aa04a6a20ca4" />

**volatile** <- 코어 캐시 기능을 무효화하고 항상 메모리를 읽도록 함.

하지만 문제는 메모리를 두 스레드가 동시에 읽을 수 있다는 것 = 가시성(Visibility) 보장은 하지만, 원자성(Atomicity)은 보장하지 않음

volatile을 사용하면 한 스레드가 값을 변경하면 즉시 다른 스레드도 그 변경을 볼 수 있지만 두 스레드가 같은 volatile 변수를 동시에 읽거나 쓰는 상황에서, 그 연산이 복합 연산(예: 읽고 수정 후 쓰기)이라면, 여전히 경쟁 조건(race condition)이 발생.
단순 읽기나 쓰기와 같은 기본적인 연산은 원자적일 수 있지만(예: int, long 등 일부 기본 타입은 자바에서 원자적이라고 보장됨), 여러 단계로 이루어진 연산은 그렇지 않음

**Atomic** 관련 클래스 <- CAS를 통해 구현해 속도도 챙긴 원자적 클래스들

CAS = Compare And Swap

<img width="568" alt="image" src="https://github.com/user-attachments/assets/5c315fd0-3dcb-4071-9c49-0104c5832c06" />

이러한 CAS는 Synchronized 보단 사용 가능한 수준이나, Spin Lock으로 인한 **Busy Wating**의 가능성이 존재한다는 단점도 있다.





