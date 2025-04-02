# 이펙티브 자바 정리

의미있게 다가왔던 부분

## **1 - 6. 객체 생성을 피하라**

new String("hello") => String s = "hello"

후자는 JVM 내에서 동일한 리터럴을 공유하여 사용

```
        Long sum = 0L;
        for (long i =0 ; i<= Integer.MAX_VALUE; i++){
            sum+=i;
        }
```

-> 5.05초

```
        long sum = 0L;
        for (long i =0 ; i<= Integer.MAX_VALUE; i++){
            sum+=i;
        }
```

-> 0.7초

Long -> long 하나만 바꿔줘도 실행시간에 대략 5초의 차이가 난다. 따라서 자바 개발자는 **의미없는 오토 박싱에 주의해야한다.** 

왜? Long sum의 경우 매 변경마다 객체가 생성된다. 메모리 + 시간 소요

## **1 - 9. try-with-resources**

```
static String firstLineOfFile(String path){
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
    return br.readline();
  } finally {
    br.close()
  }
```

```
static String firstLineOfFile(String path){
  try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readline();
  }
```

위의 코드에서 close하지 않는다면, OS 레벨의 네이티브 리소스가 남아 있게 된다. 

try-with-resource를 사용하면 코드도 짧아지고, close도 자동으로 진행해준다. 

## **5 - 28. 배열보다는 리스트를 사용하라**

왜.. 우선 배열은 공변을 허용한다. 

따라서 Object[] objects = new Long[1] 도 허용된다. 그리고 objects에 string을 넣을 때, 그 때 런타임 오류가 난다. 

반면 List는 List<Object> ol = new ArrayList<Long>(); 아예 컴파일 부터 안된다.

개발자는 **컴파일 에러가 런타임보다 훨씬 낫다**는걸 알 것이다.

```
    public static class Chooser<T> {
        private final T[] choiceList;

        public Chooser(Collection<T> choices){
            choiceList = (T[]) choices.toArray();
        }
    }
```

에러는 아니지만 Type Cast 과정에서 ClassCastException 을 만날 가능성에 경고를 보낸다.

```
    public static class Chooser<T> {
        private final List<T> choiceList;

        public Chooser(Collection<T> choices){
            choiceList = new ArrayList<>(choices);
        }

        public T choose(){
            Random rnd = ThreadLocalRandom.current();
            return choiceList.get(rnd.nextInt(choiceList.size()));
        }
    }
```

제네릭 리스트는 약간의 리소스 소모를 제외하면 런타임 예외 걱정이 없다. 충분히 투자할만 하다.

<br/>
+) 그리고 제네릭은.. T 고정이 아닌, <> 문법이 제네릭을 표현한다. T,E,K 이런건 그냥 표현용

++) 또또 제네릭은.. 컴파일 시점에만 체크하지 뒤는 그냥 동일한 원시 타입(raw type)인 List로 처리된다. 이는 제네릭이 없던 5 이전 버전과의 호환 + 약간의 성능 이득 이라고 한다. 

그냥 컴파일에만 체크 해주고, 실제 런타임에는 무관심한듯


## Enum

정수형 열거 << Enum 클래스

Enum의 Oridinal은 서비스 개발 용도에선 사용하는 것이 아니다. (EnumSet, EnumMap 성능용)

각 Enum 인스턴스 별로 구별해야하는 값이나 함수는, Switch Case 구문이 아닌 인스턴스 별 필드나 상수별 메서드 구현을 활용하자

EnumSet, EnumMap은 Enum에 특화돼있음. 속도, 효율성 측면에서 이득이 큼 (Oridinal과 배열을 사용)

<br>

## 함수형 프로그래밍

Java 8부터 지원된 람다와 스트림에 앞서 함수형 프로그래밍이 일단 뭘까?? 왜 굳이굳이 자바는 이를 지원했을까?

**객체지향**: 객체(상태와 행동을 함께 캡슐화한 단위)를 중심으로, 객체 간 메시지 전달로 협력하는 구조를 강조합니다. 데이터(필드) 변경 등 상태 변화를 자주 수반합니다.

**함수형**: 순수 함수(Pure Function)를 통해서 입력이 같으면 항상 같은 결과를 반환하도록 하며, 외부 상태를 변경하지 않도록 권장합니다. 부작용 없는 코드를 지향하므로, 병렬 처리가 쉽고 테스트가 간단합니다.


+) Stream을 사용해 명령형(for문) 대신 선언적으로 “데이터 변환 및 필터링” 로직을 표현 가능해진다.

+) 참조 투명성을 지녔기 때문에 함수형 프로그래밍은 멀티코어 프로세스에서 교착상태에 빠지지 않는다는 장점이 있다. 함수형 프로그래밍은 동시성 프로그래밍에서 강력한 프로그래밍 패러다임으로 작용한다.

없다. 그러니까 가변성을 가진 데이터가 없다. 동시성 문제에 직면하지 않는다.

내가 생각했을 때, 함수형 프로그래밍의 장점은

**1. 코드가 간결해진다.
2. 선언적 프로그래밍을 통한 가독성 증대
3. 멀티코어 환경에서 병렬 처리 및 동시성 문제 해결**
   
그래서 자바에서 변하는 시대에 흐름에 맞춰 도입했다고 생각한다.

## 익명 클래스, 람다, 메서드 참조

익명클래스 < 람다 < 메서드 참조 (대부분의 상황에서)

익명 클래스는 자리를 잃었다. 사실상 추상 클래스 구현이나 자기 참조가 필요할 때만,,

람다보단 메서드 참조이지만, 람다가 더 간결하거나, 이해하기 쉬운 수준의 코드면 람다가 나을 수도 있다. 

