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

|         Bag        |      Lsit      |       Set      |
| :----------------: | :------------: | :------------: |
|      unordered     |     ordered    |    unordered   |
|     duplicates     |   duplicates   |  no duplicates |
|   PersistenceBag   | PersistentList | PersistenceSet |
| better performance |                |                |

JPA Entity 연관관계의 Collection 은 특정 순서로 연관관계를 검색하면 데이터 베이스 쿼리 속도가 느려지기 때문에 순서에 상관없는 Bag 이라는 자료구조를 사용한다.

자바 컬렉션 프레임워크에서는 Bag이 없기 때문에 JPA에서 <mark style="color:red;">List를 Bag으로써 사용</mark>하고 있는 것이다.

### Bag vs List vs Set

Jpa에서 연관관계를 나타낼 때 Bag, List, Set은 어떤 경우에 사용하고 어떤 차이점이 있을까?

* Bag&#x20;
  * ArrayList로 초기화 중복을 허용하기 때문에 엔티티를 추가할때 비교 없이 단순히 저장만 한다. 엔티티를 추가할 때 지연로딩된 컬렉션을 초기화 하지 않는다.
* **Set**
  * HashSet으로 초기화 중복을 허용하지 않으므로 엔티티를 추가할때 비교해야 한다. 엔티티를 추가할 때 지연로딩된 컬렉션을 초기화한다.
*   **List (**순서가 있는 특수 컬렉션**)**

    {% code overflow="wrap" lineNumbers="true" %}
    ```java
    @Entity
    public class Team {

        @Id @GeneratedValue
        private Long id;

        @OneToMany(mappedBy = "team")
        @OrderColumn(name = "name")
        private List<Member> member = new ArrayList<>();

        ...
    }
    ```
    {% endcode %}
