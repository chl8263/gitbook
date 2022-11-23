# OneToOne Unique 조건에 대해



1:1 테이블 관계의 예시는 아래와 같다.

<figure><img src="../../.gitbook/assets/image (11) (3).png" alt=""><figcaption></figcaption></figure>

N:1 양방향 매핑처럼 외래 키가 있는 곳이 연관관계의 주인이다.

즉 DB 의 어떤 테이블에서 외래키를 가지고 있냐에 따라 연관관계의 주인인지 판별이 되는것.

1:1 관계의 반대는 mappedBy 를 적용하여 read only 속성으로 참조가 가능하다.

아래 예시 그림처럼 외래키가 없는 테이블인데 Entity 객체에서 참조하려한다면 연관관계 매핑 오류가 난다.

<figure><img src="../../.gitbook/assets/image (3) (2).png" alt=""><figcaption></figcaption></figure>

1:1, one to one 이라는 말은 ;외래키를 이용하여 하나의 매핑된 데이터가 있다’ 라는 말인데 그럼 당연히 외래키에 대해 unique 제약조건이 필요하고 걸어주는 것이 당연한게 아닐까?

우선 위에 대한 답은 JPA 는 @OneToOne 관계에서 외래키에 대해 unique 제약조건을 옵션으로 가지고 있다.

* `@JoinColumn(name = "MEMBER_ID", unique = true)` 유니크 제약조건을 걸음
* `@JoinColumn(name = "MEMBER_ID", unique = false)` default, 유니크 제약조건을 걸지않음

JPA 에서 OneToOne 매핑 해줬다고 강제로 제약조건이 생기는 것은 개발자가 예상치 못한 제약조건이 들어가버린 다는 뜻이되니 위와 같이 어느정도 허용은 해주는 것 이라고 생각한다.

또한 여러 DBMS 에서 unique 제약조건에 NULL 을 허용 혹은 허용하지 않는 것이 다르기 때문에 정말 명확한 1:1 관계인지 판별하고 해당 조건을 JPA 에서 사용하는 것이 필요하다고 본다.

* mysql 은 unique 제약조건에 NULL 을 허용하기 때문에 `@JoinColumn(name = "MEMBER_ID", unique = true)` 을 걸어도 다수의 null 이 들어갈 수 있음

<figure><img src="../../.gitbook/assets/image (6) (2) (1).png" alt=""><figcaption></figcaption></figure>
