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

**가장 좋은 해결책은 가변 데이터를 공유하지 않는 것이다.**


<br>

## 동기화와 성능, 교착상태

동기화 (synchronized) 내부에서 순진하게 외부 객체를 부르거나 하는 행위를 지양하자

거기에 무엇이 담겨있을 지, 얼마나 걸릴지 모른다. -> 교착상태 발생 가능성이 생기고, 블락킹 시간 증가

**따라서 동기화는 필요할 때만 간결하게 사용하고 빠르게 해제하는 것이 핵심이다.**


<br>

## 성능이 필요하다면 내부 동기화, 그 외에는 외부에서 관리

내부 동기화는 빠르지만 위험하다.

오히려 클래스나 메소드에 대해 스레드 안전하지 않다는 것을 알려주고 클라이언트에서 관리하도록 시키는 것이 좋은 경우가 많다.




