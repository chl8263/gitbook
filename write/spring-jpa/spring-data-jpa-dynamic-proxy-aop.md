# spring data jpa 동작방식과 Dynamic Proxy, AOP

## 문제 상황

Spring Data Jpa 는 Entity 를 interface 의 JpaRepository 상속 부분에 명시만 해주면 알아서 Spring 이 제공하는 Jpa 의 편리하고 다양한 기술들을 사용할 수있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

}
```

이렇게 Interface 만 적어주면 @Autowired 로 클래스를 주입받아 아래와 같이 사용할 수있는데

```java
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void testMember() {
        Member member = new Member("ewan");
        Member savedMember = memberRepository.save(member);

        Member foundMember = memberRepository.findById(savedMember.getId()).get();
        assertThat(foundMember.getId()).isEqualTo(member.getId());
        assertThat(foundMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(foundMember).isEqualTo(member);
    }
}
```

앞서 정의한 `MemberRepository` 는 Interface 가 아닌가? 어떻게 객체로 만들어져서 DI 주입을 받을 수있을까?

먼저 DI 로 주입 된 `MemberRepository` 를 디버깅을 걸어보면 아래와 같다.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

`MemberRepository` 는 Proxy 로 모양을 띄고 있고 그 하위에 `JdkDynamicAopProxy` 라는 것이 보이고 또 하위에 `advice` 라는 것이 보인다.

advice? --> Spring AOP 의 개념중 하나인데 이와 관련이 있어보인다.

## Spring Data Jpa 의 동작방식

Spring Data Jpa 는 main root 의 모든 패키지를 탐색하여 아래와 같은 `Interface` 들을 스캔한 뒤 해당 Interface 들을 인스턴스화 시킨다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
```

해당 동작 원래는 Java 의 `java.lang.reflect` package 의 `Proxy.class` 로 인해 가능한 것인데.

Spring 은 이를 Wrapping 해서 `org.springframework.aop.framework` package 의 `ProxyFactory` 를 만들었고

&#x20;`org.springframework.data.repository.core.support` package 의 `RepositoryFactorySupport`  가 `ProxyFactory` 를 이용하여 해당 작업을 진행한다.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption><p><code>RepositoryFactorySupport.class</code> 의 패키지 스캔부</p></figcaption></figure>

## 그렇다면 동작 원리는..?

java 의 DynamicProxy 를 사용하여 해당 동작을 하는데 구체적으로 어떤 원리일까?

먼저 proxy 의 개념부터 살펴보자.

### Proxy 패턴이란?

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption><p><a href="https://en.wikipedia.org/wiki/Proxy_pattern">https://en.wikipedia.org/wiki/Proxy_pattern</a></p></figcaption></figure>

같은 Real Subject 와 Proxy 는 같은 Subject(Interface) 를 구현한다.

클라이언트는 Real Subject 를 사용하는 대신 Proxy 만을 사용하고 Proxy에서 Real Subject 를 사용한다.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

뭔소린지 모르겠으니 코드를 보자.

Member Entity 의 이름을 출력하는 기능을 가지고 있는 Member service 가 있다고 가정해보자.

```java
public class MemberService {

    public void printMemberName(Member member) {
        System.out.println("Member name : " + member.getUsername());
    }
}
```

아래 Test 의 결과는 `Ewan` 이라는 이름을 가진 Member Entity의 이름을 출력한다.

```java
@Test
public void getMembernameTest() {
    Member member = new Member();
    member.setUsername("Ewan");
    member.setAge(30);
    
    MemberService memberService = new MemberService();
    memberService.printMemberName(member);
}
```

```
Member name : Ewan
```

클라이언트는 맴버 이름을 출력할 때 맴버의 나이도 함께 출력하고싶다. 그리고 맴버의 나이가 12 살 이상일 때만 이름출력을 걸기로 했.

그렇다면 위의 구조에서는 아래와 같이 변경하면 될거같다.

```java
public void printMemberName(Member member) {
    if (member.getAge() > 12) {
        System.out.println("Member age : " + member.getAge());
        System.out.println("Member name : " + member.getUsername());
    } else {
        System.out.println("To print the member's name, the member must be at least 12 years old.");
    }
}
```

분명 `printMemberName` method 는 member 의 name 을 출력하는 역할인데 저런 로직이 포함되는 것이 맞을까? 이는 단일 책임 원칙(SRP) 를 위배하는 행동이라고도 볼 수있다.

위의 문제 상황을 Proxy 패턴을 사용하여 개선해보자.



먼저 Proxy 패턴의 같은 Subject(interface)를 가져야 하는 MemberService 를 만들었다.

```java
public interface MemberService {

    public void printMemberName(Member member);
}
```

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>



그다음 RealSubject 는 아래와 같이 구성했다. 위의 Subject 를 구현하고 단순히 `printMemberName` 은 Member 의 이름만 출력할 뿐이다.

```java
public class DefaultMemberService implements MemberService{

    public void printMemberName(Member member) {
        System.out.println("Member name : " + member.getUsername());
    }
}
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>



마지막으로 Proxy 객체를 생성했다.&#x20;

proxy 객체 역시 MemberService 를 구현하고 Member Service 의 인스턴스를 생성자에서 받아 `printMemberName` 을 호출할 때 Real subject 의 `printMemberName` 를 호출하면서 Proxy 에서 부가적인 로직들도 같이 들어간다.

```java
public class MemberServiceProxy implements MemberService {

    private MemberService memberService;

    public MemberServiceProxy (MemberService memberService) {
        this.memberService = memberService;
    }

    public void printMemberName(Member member) {
        if (member.getAge() > 12) {
            System.out.println("Member age : " + member.getAge());
            memberService.printMemberName(member);
        } else {
            System.out.println("To print the member's name, the member must be at least 12 years old.");
        }
    }
}
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>



Client 는 이제 `MemberServiceProxy` 를 이용하여 아래와 같이 사용하고 원하는대로 동작할 것이다.

```java
@Test
public void getMembernameTest_proxy() {
    Member member = new Member();
    member.setUsername("Ewan");
    member.setAge(30);

    MemberService memberService = new MemberServiceProxy(new DefaultMemberService());
    memberService.printMemberName(member);
}
```

```
Member age : 30
Member name : Ewan
```



위와 같이 Proxy 패턴을 사용했을 시 아래와 같은 특징과 장점이 있다.

* 클라이언트는 프록시를 거쳐서 리얼 서브젝트를 사용하기 때문에 프록시는 리얼 서브젝트에 대한 접근을 관리거나 부가기능을 제공하거나, 리턴값을 변경할 수도 있다.&#x20;
* &#x20;리얼 서브트는 자신이 해야 할 일만 하면서(SRP) 프록시를 사용해서 부가적인 기능(접근 제한, 로깅, 트랜잭션, 등)을 제공할 때 이런 패턴을 주로 사용한다.



프록시 패턴을 사용하기 위해 매번 저런 Proxy 클래스를 만들어야 할까? Proxy 가 필요할 때마다 생성하는 것 이외에도 proxy class 의 서로 다른 method 에 같은 로직이 중복된다면..?

해당 불편함을 해소하기 위해 Java reflection 은 Dynamic Proxy 라는 기능을 제공한다.

### JDK Dynamic Proxy 란?

`Dynamic Proxy`란 이전과 같이 `Proxy` 객체를 직접 생성을 하는 것이 아니라 **Runtime(애플리케이션이 실행되는 중)에 Interface를 구현하는 Class or 인스턴스를 만들어내는 것.**

Java `java.lang.reflect.Proxy` package에서 제공해주는 API를 이용하여 `Dynamic Proxy`를 이용할 수 있다.



API에서 제공해주는 Syntax는 다음과 같다.

```java
public static Object newProxyInstance(
    ClassLoader loader, // 1
    Class<?>[] interfaces, // 2
    InvocationHandler h // 3
) throws IllegalArgumentException
```

1. `Proxy`객체 정의하기 위한 `Class Loader`를 지정한다. (`Proxy` 객체가 구현할 Interface에 `Class Loader`를 얻어오는 것이 일반적)
2. `newProxyInstance()`를 통해 생성 될 `Proxy` 객체가 구현할 Interface를 정의한다.
3. 메소드 호출을 디스패치하기 위한 호출 핸들러. (디스패치: 어떤 메소드를 호출할 것인가를 결정하여 그것을 실행하는 과정을 이야기함.)

위에 정의된 대로 매개변수를 지정하여 API를 호출하면 return 되는 값은 1번의 `Class Loader`에 정의한 타입이 반환된다.&#x20;



아래 예제 코드를 보자. 위의 Proxy 패턴에서 사용했던 class 들 중 Proxy 에 해당하는 코드를 삭제 하고 아래 코드만 남아있는 상태.

```java
public interface MemberService {    // Subject

    public void printMemberName(Member member);
}

public class DefaultMemberService implements MemberService{    // Real Subject

    public void printMemberName(Member member) {
        System.out.println("Member name : " + member.getUsername());
    }
}
```

이제 Proxy 코드 가 없으니 동적으로 Proxy 를 생성해보자.

```java
MemberService memberService = (MemberService) Proxy.newProxyInstance(MemberService.class.getClassLoader(), new Class[]{MemberService.class},
            new InvocationHandler(){

                MemberService memberService = new DefaultMemberService();    // real subject

                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    if (method.getName().equals("printMemberName")) {
                        // ... some logic
                        Object invoke = method.invoke(memberService, args);
                        // ... some logic
                        return invoke;
                    } else if (method.getName().equals("getMembersAge")) {
                        // ... some logic
                        Object invoke = method.invoke(memberService, args);
                        // ... some logic
                        return invoke;
                    }
                    return method.invoke(memberService, args);
                }
            });
```

위의 dynamic proxy 의 `InvocationHandler` 에서 `invoke` 메서드를 통해 method 를 직접 제어할수 있고 런타임에 동작하니 기존 코드보다는 유연해 보인다.

하지만 여기서 문제는 `InvocationHandler` 가 그렇게유연하지는 않다는 것이다. `invoke` 메서드에 부가적인 코드들이 들어가서 코드가 방대해 질 수있고 그렇다면 해당 dynamic proxy 를 감싸는 다른 proxy 가 만들어 져야할 수도 있다.

#### <mark style="color:blue;">Spring 에서는 위의 구조를 개선하여 Spring AOP 를 만들었고 그래서 Spring AOP 가 Proxy 기반의 AOP 라고 불리는 것이.</mark>



ref:&#x20;

[https://www.oodesign.com/proxy-pattern](https://www.oodesign.com/proxy-pattern)

[http://velog.io/@dev\_leewoooo/Dynamic-Proxy%EB%9E%80](https://velog.io/@dev\_leewoooo/Dynamic-Proxy%EB%9E%80)
