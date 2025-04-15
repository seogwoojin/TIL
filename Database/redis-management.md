# 레디스 

(이 글은 실전 Redis 책을 읽고 배운 내용을 설명하는 글입니다)

## 레디스는 싱글쓰레드?

레디스는 JS와 같이, 하나의 메인스레드가 요청을 처리하고 받은 요청을 큐에 담는다.

IO요청을 판단하고, 이를 받는 IO전용 스레드들이 존재한다. (6부터)

이제 이 스레드들이 메인 이벤트 큐에 집어넣는다 뭘? '-- 한 요청을 수행해라'

[[set key 1, Client 1], [set key 2]]

큐를 돌던, 메인 스레드는 [set key 1]부터 수행하게 된다.

명령 수행후 클라이언트 연결에 대한 정보(예: 소켓 파일 디스크립터)가 포함된 client 객체에 응답 버퍼에 응답을 담고, I/O 스레드에 이벤트를 던집니다.

이렇게 응답을 받은 I/O전용 스레드는 소켓을 통해 응답을 전달합니다.



## Redis Snapshot

레디스는 Dump RDB 파일을 통해서 스냅숏을 저장한다. 

그리고 이 스냅샷의 업데이트 기준은 다음과 같다

<img width="644" alt="image" src="https://github.com/user-attachments/assets/c028c3ad-abef-4596-baac-2e36e01c4e5b" />

- 15분에 1번 이상 변경 <- (책에서는 60분인 것으로 보아 업데이트 된듯하다)
- 5분에 10번 이상 변경
- 1분에 10000번 이상 변경 시

<img width="653" alt="image" src="https://github.com/user-attachments/assets/dbb53dcb-caa2-4435-949e-db7a0c16e96f" />

실제로 로그 파일을 보면 900초에 한번씩 업데이트 하는 것을 볼 수 있습니다.


기본적으로 SAVE 명령어를 수행 시 싱글 스레드 기반의 레디스는 다른 요청을 처리할 수 없기에, BGSAVE를 활용해 백그라운드에서 포크처리하여 덤프 파일을 생성합니다.



AWS Elasticache는 기본적으로 스냅숏을 제공하지 않습니다.

## Redis AOF

이런 식의 스냅숏은 이후의 데이터를 보관할 수 없습니다. 15분 사이에 서버가 다운된 경우 그 부분은 살려낼 수 없다는 뜻

AOF는 모든 명령어를 저장하므로 이를 보완할 수 있다. (리소스 소모는 당연히 크다, 기본적으로 비활성화)

<img width="204" alt="image" src="https://github.com/user-attachments/assets/caa3781e-f1b8-4a1f-b623-8339bf071076" />

기본설정은 매초 단위이지만 appendOnly를 yes로 바꿔주지 않으면 기본적으로 동작 X

바꿔서 한번 롤백 과정을 재현해보자

AOF를 yes로 설정하고 Redis를 재시작 해보면 새로운 aof 파일을 볼 수 있다.

<img width="520" alt="image" src="https://github.com/user-attachments/assets/5c000810-93f5-4040-8032-305e0cabd66f" />

```
*2
$6
SELECT
$1
0
*3
$3
set
$4
key2
$5
key23
*3
$3
set
$4
key1
$6
keynet
```

aof 파일 내부

만약 되살려야 하는 상황이 온다면, 모든 과정을 **무식하게** 반복해야함. CPU 소모 및 시간 오래걸림

## 실전에서 수행 전략 

1. RDB만 사용

캐시 서버는 캐시서버 답게, 조금은 날아가도 크리티컬하지 않다면, 굳이 리소스를 소모해가며 AOF를 사용하지 말자

2. RDB + AOF 일부

aof-use-rdb-preamble yes를 통해서 AOF가 새로이 저장되는 시점의 RDB를 체크포인트로 삼아서 수정합니다.

데이터 유실이 크리티컬한 상황에서 고려할 수 있습니다.



