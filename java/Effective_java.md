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
