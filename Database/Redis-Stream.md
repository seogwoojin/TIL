# Redis Stream vs Redis Pub/Sub

실전 Redis 라는 책을 읽으며 알게된 내용 중, Redis 만의 Pub/Sub, Stream 자료형에 대해 남기고자 작성합니다.

레디스를 단순한 캐시서버라고 생각했는데 실제로 꽤 다양한 기능을 제공하고 있었고,

채팅 서버 구축에도 사용 가능하거나 지리적 계산 Sorted Set을 통한 랭킹 시스템 구축에도 사용된다는 것을 알 수 있었습니다.

<br>

그중에서도 Redis의 Pub Sub과 Stream을 비교하며, 간단한 채팅 서버를 구축했습니다.

### Redis Pub/Sub

![image](https://github.com/user-attachments/assets/8b5ea3af-f6cd-4b00-a3b2-a2b52b94239a)

우선 더 단순한 Pub / Sub 구조부터 설명하겠습니다. 

```
SUBSCRIBE mychannel1 mychannel2

~~~

PUBLISH mychannel2 "hello, world"
```

와 같은 단순한 작업으로 구독과 발행을 구현할 수 있습니다.

레디스 Pub/Sub 기능의 주의할 점은 과거 발행 메시지는 받을 수 없다는 점.

이는 끊기거나 하는 시점에서 재접속해도 중간의 메세지는 받을 수 없음을 의미합니다. 즉 이런 점을 지키지 않아도 되는 상황에서 사용하거나

Redis 5.0 이상 버전 부터는 **Redis Stream**을 사용하는 편이 낫습니다.

