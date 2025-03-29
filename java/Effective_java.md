# 이펙티브 자바 정리

의미있게 다가왔던 부분

**1 - 6. 객체 생성을 피하라**

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

**1 - 9. try-with-resources**

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

**5 - 28. 배열보다는 리스트를 사용하라**

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


