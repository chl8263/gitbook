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
List<Team> team3 = entityManager.createQuery(
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

### Bag vs List vs Set

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
    *
