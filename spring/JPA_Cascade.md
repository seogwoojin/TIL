# `JPA_Cascade`

JPA를 활용하며 1:N, 1:1, N:1 에서 Casecade Type을 쓰는 경우가 종종 있습니다.

## Cascade (영속성 전이) 종류

Persist -> 영속

Remove -> 삭제

All -> 영속 + 삭제

Merge -> 병합

Refresh, Detach -> 사용 X

이중 중요하게 봐야할 것은 Persist, All 입니다. (보통 Remove를 쓰는 경우면 Persist도 사용하기 때문)

<br>

```
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long bno;

    @Column
    private String title;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;
}

public class Member {
    @Column
    private String name;

    // 영속성 전이
    @OneToMany(cascade = CascadeType.PERSIST, mappedBy = "member")
    private List<Board> boards;
}
```

Persist는 영속성 컨텍스트에서 주인 객체가 반영될때, 연관된 객체들도 함께 반영되는 것을 의미합니다.

따라서 예시에 따르면 Member 객체가 반영될 때, boards에 있는 Board 객체들도 DB에 반영되게 됩니다. 따라서 따로 Board들의 저장 로직을 만들지 않아도 돼서 편리합니다.

Remove도 비슷하지만 이번엔 삭제가 반영된다는 점. 만약 boards 라는 List에서 기존 Board 객체를 제거한 후 반영하면, 그 Board가 DB에서 Delete 됩니다.

ALL은 Persist + Remove 입니다.

<br>

## 써도 되는 경우

`자식 객체가 부모 객체와 동일한 생애 주기를 가질 때` 사용합니다.

예를 들어, MemberTag 라는 객체를 새로 만들어 보겠습니다.

```
public class Member {
    @Column
    private String name;

    // 영속성 전이
    @OneToMany(cascade = CascadeType.PERSIST, mappedBy = "member")
    private List<MemberTag> tags;
}

public class MemberTag {
    @Column
    private String tags;

    // 영속성 전이
    @ManyToOne
    @JoinColumn(name = "member_id")
    private Member member;
}
```

Member가 없어지면, 사실상 MemberTag가 고아 객체가 되기 때문에, 이 경우, CascadeType.ALL을 거는 것이 좋은 생각일 수 있습니다.

<br>

## `쓰면 안되는 경우`

```
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long bno;

    @Column
    private String title;

    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;

    @ManyToOne(fetch = FetchType.LAZY)
    private Advertise advertise;
}

public class Member {
    @Column
    private String name;

    // 영속성 전이
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "member")
    private List<Board> boards;
}
```

- `자식 객체가 다른 엔티티와 연관을 맺고 있는 경우`
- `자식 객체가 부모 객체가 없더라도 생존해야 하는 경우`


만약 Board에 광고와 같은 다른 엔티티와 연관 관계가 있다면, 그 연관 엔티티 때문에 원하는 대로 동작이 이루어지지 않거나, 

객체 간의 싱크가 맞지 않는 경우가 발생할 수 있습니다. 

<br>

영속성과 관련하여 곤란한 상황이 발생할 수 있기 때문에 Cascade 전략은 User - UserInfo 처럼 확실한 경우가 아니라면, 자제하는 게 좋을 것 같다.


