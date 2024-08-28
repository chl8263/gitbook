# 왜 save 후 update 할 때 query가 두번 실행되는가?

JPA 에서 아래와 같이 Entity 생성후 entity manager 에 persist 를 등록해 주고 transaction commit 혹은 flush 를 진행하면 insert query 가 DB 에 나가게 된다.

```java
Member member = new Member();
member.setId(1L);
member.setName("Ewan");

entityManager.persist(member);

```

```
Hibernate: 
    /* insert hellojpa.Member
        */ insert 
        into
            Member
            (name, id) 
        values
            (?, ?)
```

persist 에 entity 를 등록시킨 후 entity를 직접 수정만 하면 transaction commit 혹은 flush 시점에 JPA는 update query 를 만들어 DB에 요청하는데

```
Hibernate: 
    /* update
        hellojpa.Member */ update
            Member 
        set
            name=? 
        where
            id=?
```

엔티티가 만들어지고 수정만 한번 하면 되니까 update 쿼리를 굳이 보내지 않고 수정된 내용으로 insert 쿼리를 한번만 날리면 되지 않을까?

먼저 JPA 의 구조는 아래와 같다.

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

먼저, persist하게 되면 1차캐시에 저장하고 스냅샷을 보관한다. 그리고 insert 쿼리를 생성하여 쓰기 지연 SQL 저장소에 보관을 하게된다.

이제 Member는 영속성 컨텍스트에 의해 관리되고 있는 상태이고 이 상태에서 Member entity의 값을 변경하게 되면 트랜잭션 커밋이 진행되면서 플러시가 호출되고 변경된 점을 찾아 update쿼리를 생성하여 쓰기 지연 SQL 저장소에 보관하게된다.

이 시점에서 쓰기 지연 SQL 저장소를 들여다보면 생성된 insert, 그 후에 변경감지에 의해 생성된 update 쿼리 총 2개가 쌓여있으므로 이 2개가 DB로 전송되면서 insert, update 쿼리가 발생하게된다.

### JPA 내부적으로 어떻게 처음 entity 상태 스냅샷을 관리할까?

우선 `entityManager.persist(member);` 로 entity를 영속성 상태로 등록하게되면

entity manager 의 session 내부적으로 persistenceContext 의 entityEntryContext 에 Linked List 와 유사한 방식으로 해당 트랜잭션 내의 entity 들을 관리하게 된다.

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

entityEntryContext 의 ManagedEntity 들은 아래 그림과 같이 처음 load 된 상태(아래 그림의 loadedState) 의 Entity 의 ID 를 제외한 맴버 값을 가지고 있다.

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

Flush 를 하는 시점에서 Session(Entity Manager) 의 ActionQueue 에 현재 entity와 load 된 entity 의 값을 비교하여 update List 에 해당 부분을 저장하게 된다.

여기서 ActionQueue 란 Entitymanager 가 해당 트랜잭션에서 수행되는 insert, update, delete 등 을 감지하여 알맞은 List 에 넣어놓게된다.&#x20;

실제로 JPA 가 최종적으로 Query 를 만들 때 ActionQueue 를 참조하여 수행된다.

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

위의 예제 에서는 persist(Member) 를 할때 insert List 에 삽입이되고, flush 할때 관리되는 Entity의 변경이 있을시 update List 에 저장되어 최종적으로 query 를 만들 때 insert 한번, update 한번이 DB에 날아가게 되는것.
