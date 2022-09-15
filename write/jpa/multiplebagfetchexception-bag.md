# MultipleBagFetchException 과 Bag

## 문제 상황

JPA 사용 시 아래 에러를 마주치는 상황이 온다.

{% code overflow="wrap" %}
```
Caused by: org.hibernate.loader.MultipleBagFetchException: cannot simultaneously fetch multiple bags: [hellojpa.Team.member, hellojpa.Team.account]
...
```
{% endcode %}

문제 상황은 크게 아래 두개의 상황에서 발생한다.

* 하나의 Entity 에 `to Many` 관계가 두개 이상이고 Eager 로 설정되어 있는 경우&#x20;

{% code overflow="wrap" lineNumbers="true" %}
```java
@Entity
public class Team {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "TEAM_ID")
    private Long id;

    @OneToMany(mappedBy = "team", fetch = FetchType.EAGER)
    private List<Member> member = new ArrayList<>();

    @OneToMany(mappedBy = "team", fetch = FetchType.EAGER)
    private List<Account> account = new ArrayList<>();
    
    ....
```
{% endcode %}

* 하나의 Entity 에 `to Many` 관계가 두개 이상이고 Lazy 로 설정되어 있는 상황에서, 해당 Entity에 두개이상의 `to Many` 관계를 Fetch Join 하려할 때

{% code overflow="wrap" lineNumbers="true" %}
```java
@Entity
public class Team {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "TEAM_ID")
    private Long id;

    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private List<Member> member = new ArrayList<>();

    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private List<Account> account = new ArrayList<>();
    
    ...
```
{% endcode %}

{% code overflow="wrap" lineNumbers="true" %}
```java
List<Team> team = entityManager.createQuery(
                    "SELECT t " + 
                      "FROM Team t " +
                      "JOIN FETCH t.member " +
                      "JOIN FETCH t.account")
                      .getResultList();
```
{% endcode %}

## 원인 분석

해당 에러는 왜 나는것이며 문제 상황에서 발생하는 `MultipleBagFetchException` 의 Bag 이란 무엇일까?

> A generalization of the notion of a set is that of a multiset or bag, which is similar to a set but allows repeated (“equal”) values (duplicates).\
> [https://en.wikipedia.org/wiki/Set\_(abstract\_data\_type)#Multiset](https://en.wikipedia.org/wiki/Set\_\(abstract\_data\_type\)#Multiset)

> A Bag is a java collection that stores elements without caring about the sequencing, but allow duplicate elements in the list.\
> A bag is a random grouping of the objects in the list.\
> [https://en.wikipedia.org/wiki/Set\_(abstract\_data\_type)#Multiset](https://en.wikipedia.org/wiki/Set\_\(abstract\_data\_type\)#Multiset)

> A is an unordered collection, which can contain duplicated elements.\
> That means if you persist a bag with some order of elements, you cannot expect the same order retains when the collection is retrieved.\
> There is not a “bag” concept in Java collections framework, so we just use a java.util.List corresponds to a .\
> [https://stackoverflow.com/questions/13812283/difference-between-set-and-bag-in-hibernate](https://stackoverflow.com/questions/13812283/difference-between-set-and-bag-in-hibernate)

Bag(Multiset)은 Set과 같이 순서가 없고, List와 같이 중복을 허용하는 자료구조이다.

|     Bag    |    Lsit    |      Set      |
| :--------: | :--------: | :-----------: |
|  unordered |   ordered  |   unordered   |
| duplicates | duplicates | no duplicates |

| 컬렉션 인터페이스           | 내장 컬렉션         | 중복 허용 | 순서 보관 |
| ------------------- | -------------- | ----- | ----- |
| Collection, List    | PersistenceBag | O     | X     |
| Set                 | PersistenceSet | X     | X     |
| List + @OrderColumn | PersistentList | O     | O     |

JPA Entity 연관관계의 Collection 은 특정 순서로 연관관계를 검색하면 데이터 베이스 쿼리 속도가 느려지기 때문에 순서에 상관없는 Bag 이라는 자료구조를 사용한다.

자바 컬렉션 프레임워크에서는 Bag이 없기 때문에 엔티티를 영속 상태로 만들 때 **컬렉션 필드를 하이버네이트에서 제공하는 컬렉션(PersistentBag (**<mark style="color:red;">List를 Bag으로 사용</mark>**)) 으로 Wrapping 해서 사용한다.**

### Bag vs Set vs List&#x20;

Jpa에서 연관관계를 나타낼 때 Bag, List, Set은 어떤 경우에 사용하고 어떤 차이점이 있을까?

* Bag&#x20;
  * ArrayList로 초기화 중복을 허용하기 때문에 엔티티를 추가할때 비교 없이 단순히 저장만 한다. 엔티티를 추가할 때 지연로딩된 컬렉션을 초기화 하지 않는다.
* **Set**
  * HashSet으로 초기화 중복을 허용하지 않으므로 엔티티를 추가할때 비교해야 한다. 엔티티를 추가할 때 지연로딩된 컬렉션을 초기화한다.
  * Many to Many 일 때 유리하다. (하지만 회피 전략으로 사용하지 않)
*   **List (**순서가 있는 특수 컬렉션**)**

    {% code overflow="wrap" lineNumbers="true" %}
    ```java
    @Entity
    public class Team {

        @Id @GeneratedValue
        private Long id;

        @OneToMany(mappedBy = "team")
        @OrderColumn(name = "POSITION") // List의 위치 값을 테이블의 POSITION 컬럼에 보관. 일대다 관계여서 다쪽에 저장
        private List<Member> member = new ArrayList<>();

        ...
    }
    ```
    {% endcode %}

    * 장점보단 단점이 많아 실무에선 사용하지 않는다.
      * member 를 INSERT할 때 POSITION 값이 저장되지 않는다. @OrderColumn을 Team  엔티티에 매핑하기 때문에 Team .member 의 위치값을 사용해서 POSITION값을 UPDATE 하는 SQL이 추가로 발생한다.
      * member 를 삭제 또는 위치 변경시 연관된 많은 위치 값을 변경해야 한다.
      * 중간에 POSITION 값이 없으면 조회한 리스트에 null이 보관된다. > 컬렌션을 순회할때 NullPointerException이 발생한다.

### 2개 이상의 Many 연관관계를 Fetch Join 하면 안되는 이유

Fetch Join 과 연관관계 eager 설정 모두 결국 두개 이상의 Join 이 설정 되는 것이다.

아래 Team Entity 에서 Many 연관관계인 `member` 와 `address` 를 Fetch Join 한다고 가정 해보자.

{% code overflow="wrap" lineNumbers="true" %}
```java
@Entity
public class Team {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "TEAM_ID")
    private Long id;

    @OneToMany(mappedBy = "team")
    private List<Member> member = new ArrayList<>();

    @OneToMany(mappedBy = "team")
    private List<Address> address = new ArrayList<>();
    
    ....
```
{% endcode %}

아래코드와 같이 `Team 하나`에 `2개의 Member`와 `3개의 Address`를 연관관계 매핑을 해주었다.

```java
Team team = new Team();

Member member = new Member();
Member member1 = new Member();

Address address = new Address();
Address address2 = new Address();
Address address3 = new Address();

member.setTeam(team);
member1.setTeam(team);

address.setTeam(team);
address2.setTeam(team);
address3.setTeam(team);

entityManager.persist(team);
entityManager.persist(member);
entityManager.persist(member1);

entityManager.persist(address);
entityManager.persist(address2);
entityManager.persist(address3);
```

{% code overflow="wrap" lineNumbers="true" %}
```java
List<Team> team = entityManager.createQuery(
                    "SELECT t " + 
                      "FROM Team t " +
                      "JOIN FETCH t.member " +
                      "JOIN FETCH t.account")
                      .getResultList();
```
{% endcode %}

해당 JPQL 에서 기대하는 SQL 은 대략 아래와 같을것이다. (위의 JPQL 을 실행 하면 duplicate bag 에러 발)

```sql
SELECT team.*
     , address.*
     , member.*
  FROM Team as team
  LEFT JOIN Address as address
    ON team.TEAM_ID = address.team_TEAM_ID 
  LEFT JOIN Member as member
    ON team.TEAM_ID = member.TEAM_ID 
 WHERE team.TEAM_ID = 1
```

SQL 만 실행했을 때는 아래와 같이  총 6개의 ROW 결과가 나온다.

결국 LEFT JOIN 과 FK 로 인해 Cartesian Product (N x M) 해당 릴레이션의 조합 가능한 모든 릴레이션을 구하기 위한 집합연산이 발생한.

| TEAM\_ID | ADDRESS\_ID | createTime | updateTime | pw | userName | team\_TEAM\_ID | MEMBER\_ID | createTime | updateTime | pw | userName | TEAM\_ID |
| -------- | ----------- | ---------- | ---------- | -- | -------- | -------------- | ---------- | ---------- | ---------- | -- | -------- | -------- |
| 1        | 1           |            |            |    |          | 1              | 1          |            |            |    |          | 1        |
| 1        | 1           |            |            |    |          | 1              | 2          |            |            |    |          | 1        |
| 1        | 2           |            |            |    |          | 1              | 1          |            |            |    |          | 1        |
| 1        | 2           |            |            |    |          | 1              | 2          |            |            |    |          | 1        |
| 1        | 3           |            |            |    |          | 1              | 1          |            |            |    |          | 1        |
| 1        | 3           |            |            |    |          | 1              | 2          |            |            |    |          | 1        |

위의 DB에서 나온 6개의 결과값을 아래 `Bag` Collection 으로 설정된 연관관계 컬렉션에 어떻게 매핑할 수있을까?

{% code overflow="wrap" lineNumbers="true" %}
```java
@OneToMany(mappedBy = "team")
private List<Member> bere =new ArrayList<>();

@OneToMany(mappedBy = "team")
private List<Address> address = new ArrayList<>();
```
{% endcode %}

중복도 보장이 안 되고, 순서도 보장이 안되는 Bag 자료구조들을 매핑할 방법이 있을까?  만약 Bag 컬렉션이 더욱 더 많아진다면 Cartesian Product 로 인한 무수히 많은 Row 가 나올 것이고, 해당 ROW 를 어떤 기준으로 Bag 자료구조에 넣을 수없다.

<mark style="color:red;">**즉, 두가지 이상의 Bag 컬렉션을 이용한 Fetch Join 진행 시 Bag 은 순서가 없기 때문에 Hibernate는 올바른 열을 올바른 엔티티에 매핑 할 수 없다.**</mark>

> #### JPA에서 Fetch Join은 아래와 같은 특징을 가지고 있다.
>
> * ToOne 관계에서는 몇개든 사용이 가능하다.
> * ToMany 관계에서는 1개만 사용이 가능하다.

### 2개 이상의 Many 연관관계 Fetch Join 시 Set 을 사용한다면?

그렇다면, Fetch Join 시 Bag 컬렉션 말고 Set 을 사용하면 어떨까?

sc아래와같이 `1개의 Bag` 컬렉션과 `2개의 Set` 컬렉션을 Fetch Join(아래 예제에서 Eager) 하면 어떨까?

{% code overflow="wrap" lineNumbers="true" %}
```java
@Entity public class Team {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "TEAM_ID")
    private Long id;
    
    @OneToMany(mappedBy = "team", fetch = FetchType.EAGER)
    private Set<Member> member = new HashSet<>();

    @OneToMany(mappedBy = "team", fetch = FetchType.EAGER)
    private List<Address> address = new ArrayList<>();

    @OneToMany(mappedBy = "team", fetch = FetchType.EAGER)

    private Set<Account> account = new HashSet<>();
    
....
```
{% endcode %}

Team 하나에 `2개의 Member`, `3개의 Account`, `3개의 Address` 데이터가 있다고 가정하자.

Team 을 아래와 같이 조회해보면

<pre class="language-java" data-overflow="wrap" data-line-numbers><code class="lang-java"><strong>Team foundTeam = entityManager.find(Team.class, 1L);</strong></code></pre>

아래와 같은 쿼리가 날아가는데, Team 에는 `2개의 Member`, `3개의 Account`, `3개의 Address`  의 연관관계가 있기 때문에 18(2 x 3 x 3) 개의 데이터가 Cartesian Product 연산으로 쿼리가 불리어지게된다.

```
Hibernate: 
    select
        team0_.TEAM_ID as TEAM_ID1_9_0_,
        account1_.TEAM_ID as TEAM_ID6_0_1_,
        account1_.Account_ID as Account_1_0_1_,
        account1_.Account_ID as Account_1_0_2_,
        account1_.createTime as createTi2_0_2_,
        account1_.updateTime as updateTi3_0_2_,
        account1_.pw as pw4_0_2_,
        account1_.TEAM_ID as TEAM_ID6_0_2_,
        account1_.userName as userName5_0_2_,
        address2_.team_TEAM_ID as team_TEA6_1_3_,
        address2_.ADDRESS_ID as ADDRESS_1_1_3_,
        address2_.ADDRESS_ID as ADDRESS_1_1_4_,
        address2_.createTime as createTi2_1_4_,
        address2_.updateTime as updateTi3_1_4_,
        address2_.pw as pw4_1_4_,
        address2_.team_TEAM_ID as team_TEA6_1_4_,
        address2_.userName as userName5_1_4_,
        member3_.TEAM_ID as TEAM_ID6_6_5_,
        member3_.MEMBER_ID as MEMBER_I1_6_5_,
        member3_.MEMBER_ID as MEMBER_I1_6_6_,
        member3_.createTime as createTi2_6_6_,
        member3_.updateTime as updateTi3_6_6_,
        member3_.pw as pw4_6_6_,
        member3_.TEAM_ID as TEAM_ID6_6_6_,
        member3_.userName as userName5_6_6_ 
    from
        Team team0_ 
    left outer join
        Account account1_ 
            on team0_.TEAM_ID=account1_.TEAM_ID 
    left outer join
        Address address2_ 
            on team0_.TEAM_ID=address2_.team_TEAM_ID 
    left outer join
        Member member3_ 
            on team0_.TEAM_ID=member3_.TEAM_ID 
    where
        team0_.TEAM_ID=?

```

**결과 :**

Set Collection 으로 이루어진 Member Entity -> 2개&#x20;

Set Collection 으로 이루어진 Account Entity -> 3개&#x20;

Bag Collection 으로 이루어진 Address Entity -> 18개&#x20;

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Set Collection 인 것들은 중복을 제거하기 때문에 해당 테이블에 있는 Row 갯수만큼 가져오지만 Bag Collection 인 것은 쿼리의 Row 수 만큼 반환하는 것을 볼 수있다.

해당 작업은 정상동작을 하지만 JPA 관점에서 Team 객체의 객체 그래프를 탐색 관점에서 바라볼 때 Team 의 Address 값이 저렇게 들어가있는것이 옳은 것일까?

따라서 **단일 JPQL 쿼리에서 동시에 두 개의 컬렉션을 가져오는 경우 접근 방식이** Cartesian Product**을 생성하는 성능적인 문제와 객체 관점의 데이터 정합성 문제가 있기 때문에 상황에 맞게 Set 과 Bag 컬렉션을 사용해야한다.**
