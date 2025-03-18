# Redis Stream vs Redis Pub/Sub

실전 Redis 라는 책을 읽으며 알게된 내용 중, Redis 만의 Pub/Sub, Stream 자료형에 대해 남기고자 작성합니다.

레디스를 단순한 캐시서버라고 생각했는데 실제로 꽤 다양한 기능을 제공하고 있었고,

채팅 서버 구축에도 사용 가능하거나 지리적 계산 Sorted Set을 통한 랭킹 시스템 구축에도 사용된다는 것을 알 수 있었습니다.

<br>

그중에서도 Redis의 Pub Sub과 Stream을 비교하며, 간단한 채팅 서버를 구축했습니다.

### Redis Pub/Sub
