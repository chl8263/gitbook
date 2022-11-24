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

<figure><img src="../../.gitbook/assets/image (3) (1).png" alt=""><figcaption><p><code>RepositoryFactorySupport.class</code> 의 패키지 스캔부</p></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (5) (2).png" alt=""><figcaption></figcaption></figure>



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
MemberService memberService = 
    (MemberService) Proxy.newProxyInstance(MemberService.class.getClassLoader()
    , new Class[]{MemberService.class}
    , new InvocationHandler(){
        
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

#### <mark style="color:blue;">Spring 에서는 위의 구조를 개선하여 Spring AOP 를 만들었고 그래서 Spring AOP 가 Proxy 기반의 AOP 라고 불리는 것이다.</mark>



## Spring AOP 란?

Dynamic proxy 를 다시 정리해 보면 아래와 같다.

* 다이내믹 프록시는 프록시 팩토리에 의해 런타임 시 다이내믹하게 만들어지는 오브젝트다. 다이내믹 프록시 오브젝트는 타깃의 인터페이스와 같은 타입으로만들어진다.
* 클라이언트는 다이내믹 프록시 오브젝트를 타깃 인터페이스를 통해 사용할 수 있다.
* 프록시를 만들 때 인터페이스를 모두 구현해가면서 클래스를 정의하는 수고를 덜 수 있다.
* 프록시 팩토리에게 인터페이스 정보만 제공해주면 해당 인터페이스를 구현한 클래스의 오브젝트를 자동으로 만들어준다.
* 프록시로서 필요한 부가기능 제공 코드는 직접 작성해야 한다. 부가기능은 프록시 오브젝트와 독립적으로 InvocationHandler를 구현한 오브젝트에 담는다.
* InvocationHandler 인터페이스는 아래 메소드 하나만 가진 간단한 인터페이스이다.

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable;
```

* invoke( ) 메소드는 리플렉션의 Method 인터페이스를 파라미터로 받는다. 메소드를 호출할 때 전달되는 파라미터도 args로 받는다.
* 다이내믹 프록시 오브젝트는 클라이언트의 모든 요청을 리플렉션 정보로 변환해서 InvocationHandler 구현 오브젝트의 invoke() 메소드로 넘기는 것이다. 타깃 인터페이스의 모든 메소드 요청이 하나의 메소드로 집중되기 때문에 중복되는 기능을 효과적으로 제공할 수 있다.
* Hello 인터페이스를 제공하면서 프록시 팩토리에게 다이내믹 프록시를 만들어달라고 요청하면 Hello 인터페이스의 모든 메소드를 구현한 오브젝트를 생성해준다. InvocationHandler 인터페이스를 구현한 오브젝트를 제공해주면 다이내믹 프록시가받는 모든 요청을 InvocationHandler의 invoke() 메소드로 보내준다. Hello 인터페이스의 메소드가 아무리 많더라도 invoke() 메소드 하나로 처리할 수 있다.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Dynamic proxy 의 확장

* InvocationHandler 방식의 또 한 가지 장점은 타깃의 종류에 상관없이도 적용이 가능하다는 점이다. 어차피 리플렉션의 Method 인터페이스를 이용해 타깃의 메소드를 호출하는 것이니 Hello 타입의 타깃으로 제한할 필요도 없다.

```java
public class UppercaseHandler implements InvocationHandler {

    private final Object target;

    public UppercaseHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Object result =  method.invoke(target, args);
        if (result instanceof String) {
            return ((String) result).toUpperCase();
        }
        return result;
    }
}
```

* InvocationHandler는 단일 메소드에서 모든 요청을 처리하기 때문에 어떤 메소드에 어떤 기능을 적용할지를 선택하는 과정이 필요할 수도 있다.
* 호출하는 메소드의 이름, 파라미터의 개수와 타입, 리턴 타입 등의 정보를 가지고 부가적인 기능을 적용할 메소드를 선택할 수 있다.

### Dynamic proxy 의 한계

* 다이내믹 프록시 오브젝트는 일반적인 스프링의 빈으로는 등록할 방법이 없다. 스프링의 빈은 기본적으로 클래스 이름과 프로 퍼티로 정의된다. 스프링은 지정된 클래스 이름을 가지고 리플렉션을 이용해서 해당 클래스의 오브젝트를 만든다.
* 스프링은 내부적으로 리플렉션 API를 이용해서 빈 정의에 나오는 클래스 이름을 가지고 빈 오브젝트를 생성한다. 문제는 다이내믹 프록시 오브젝트는 이런 식으로 프록시 오브젝트가 생성되지 않는다는 점이다. 사실 다이내믹 프록시 오브젝트의 클래스가 어떤 것인지 알 수도 없다. 클래스 자체도 내부적으로 다이내믹하게 새로 정의해서 사용하기 때문이다.
* 사전에 프록시 오브젝트의 클래스 정보를 미리 알아내서 스프링의 빈에 정의할 방법이 없다.

### Dynamic proxy 의 한계에 대한 대안 - 스프링의 프록시 팩토리 빈

**다이내믹 프록시를 만들어주는 팩토리 빈**

```java
public interface FactoryBean<T> {
	@Nullable
	T getObject() throws Exception;
	@Nullable
	Class<?> getObjectType();
	default boolean isSingleton() {
		return true;
	}
}
```

* Proxy의 newProxylnstance() 메소드를 통해서만 생성이 가능한 다이내믹 프록시 오브젝트는 일반적인 방법으로는 스프링의 빈으로 등록할 수 없다.&#x20;
* 대신 팩토리 빈을 사용하면 다이내믹 프록시 오브젝트를 스프링의 빈으로 만들어줄 수가 있다.



### ProxyFactoryBean **of Spring** <a href="#41-proxyfactorybean" id="41-proxyfactorybean"></a>

* 위의 **팩토리 빈 또한 한계점 때문 Spring 에서  Wrapping 하여 나온 대안**
* 스프링은 프록시 오브젝트를 생성해주는 기술을 추상화한 팩토리 빈을 제공해준다. 팩토리 빈 이름은 ProxyFactoryBean으로 프록시를 생성해서 빈 오브젝트로 등록하게 해주는 역할을 한다.
* ProxyFactoryBean은 순수하게 프록시를 생성하는 작업만을 담당하고 프록시를 통해 제공해줄 부가기능은 별도의 빈에 둘수 있다.
* ProxyFactoryBean이 생성하는 프록시에서 사용할 부가기능은 Methodlnterceptor 인터페이스를 구현해서 만든다. Methodlnterceptor는 InvocationHandler와 비슷하지만 한 가지 다른 점이 있다.
  * InvocationHandler의 invoke() 메소드는 타깃 오브젝트에 대한 정보를 제공하지 않는다. 따라서 타깃은 InvocationHandler를 구현한 클래스가 직접 알고 있어야 한다.
  * Methodlnterceptor의 invoke() 메소드는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지도 함께 제공받는다.
* Methodlnterceptor는 타깃 오브젝트에 상관없이 독립적으로 만들어질 수 있다.

```java
@Test
public void proxyFactoryBean() {
    ProxyFactoryBean pfBean =new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());
    pfBean.addAdvice(new UpperCaseAdvice());

    Hello proxiedHello = (Hello) pfBean.getObject();
    assertThat(proxiedHello.sayHi("gunju"), is("HI GUNJU"));
}

private class UpperCaseAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Object result = invocation.proceed();
        if (result instanceof String) {
            return ((String) result).toUpperCase();
        }
        return result;
    }
}
```



#### **어드바이스: 타깃이 필요 없는 순수한 부가기능**

* Methodlnterceptor를 구현한 UppercaseAdvice에는 타깃 오브젝트가 등장하지 않는다. Methodlnterceptor로는 메 소드 정보와 함께 타깃 오브젝트가 담긴 Methodlnvocation 오브젝트가 전달된다.
* Methodlnvocation은 일종의 콜백 오브젝트로, proceed() 메소드를 실행하면 타깃 오브젝트의 메소드를 내부적으로 실행해주는 기능이 있다.
* ProxyFactoryBean은 작은 단위의 템플릿/콜백 구조를 응용해서 적용했기 때문에 템플릿 역할을 하는 Methodlnvocation을 싱글톤으로 두고 공유할 수 있다.
* addAdvice() 메소드를 통해 ProxyFactoryBean에는 여러 개의 Methodlnterceptor를 추가할 수 있다. ProxyFactoryBean 하나만으로 여러 개의 부가 기능을 제공해주는 프록시를 만들 수 있다는 뜻이다.
* Methodlnterceptor는 Advice 인터페이스를 상속하고 있는 서브인터페이스이다.
* 타깃 오브젝트에 적용하는 부가기능을 담은 오브젝트를 스프링에서는 어드바이스라고 부른다.
* ProxyFactoryBean은 인터페이스 타입을 제공받지도 않는다. ProxyFactoryBean에 있는 인터페이스 자동검출 기능을 사용해 타깃 오브젝트가 구현하고 있는 인터페이스 정보를 알아낸다. 그리고 알아낸 인터페이스를 모두 구현하는 프록시를 만들어준다.
  * 타깃 오브젝트가 구현하는 인터페이스 중에서 일부만 프록시에 적용하기를 원한다면 인터페이스 정보를 직접 제공해줘도 된다.
* ProxyFactoryBean은 기본적으로 JDK가 제공하는 다이내믹 프록시를 만들어준다. 경우에 따라서는 CGLib이라고 하는 오픈소스 바이트코드 생성 프레임워크를 이용해 프록시를 만들기도 한다.
* 재사용 가능한 기능을 만들어두고 바뀌는 부분(콜백 오브젝트와 메소드 호출정보)만 외부에서 주입해서 이를 작업 흐름(부가 기능 부여) 중에 사용하도록 하는 전형적인 템플릿/콜백 구조다.

#### **포인트컷: 부가기능 적용 대상 메소드 선정 방법**

* 기존방식

![](https://gunju-ko.github.io/assets/img/posts/toby-spring/aop/AOP-3.4.png)

* 문제점
  * 타깃이 다르고 메소드 선정 방식이 다르다면 InvocationHandler 오브젝트를 여러 프록시가 공유할 수 없다.
*   스프링의 ProxyFactoryBean 방식은 두 가지 확장 기능인 부가기능과 메소드 선정 알고리즘을 활용하는 유연한 구조를 제공한다.

    * 어드바이스 : 부가기능을 제공
    * 포인트 컷 : 메소드 선정 알고리즘, Pointcut 인터페이스를 구현해서 만듬

    ![](https://gunju-ko.github.io/assets/img/posts/toby-spring/aop/AOP-3.4.1.png)

```java
@Test
public void proxyFactoryBean() {
    ProxyFactoryBean pfBean = new ProxyFactoryBean();
    pfBean.setTarget(new HelloTarget());

    NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
    pointcut.setMappedName("sayH*");
    pfBean.addAdvisor(new DefaultPointcutAdvisor(pointcut, new UpperCaseAdvice()));

    Hello proxiedHello = (Hello) pfBean.getObject();
    assertThat(proxiedHello.sayHi("gunju"), is("HI GUNJU"));
    assertThat(proxiedHello.sayThankYou("gunju"), is("Thank you gunju"));
}
```

* 어드바이스와 포인트컷을 묶은 오브젝트를 인터페이스 이름을 따서 어드바이저라고 부른다.
* 어드바이저 = 포인트컷(메소드 선정 알고리즘)+ 어드바이스(부가 기능)



**어드바이스와 포인트컷의 재사용**

* ProxyFactoryBean은 스프링의 DI와 템플릿/콜백 패턴, 서비스 추상화 등의 기법이 모두 적용된 것이다. 그 덕분에 독립적이며 여러 프록시가 공유할 수 있는 어드바이스와 포인트컷으로 확장 기능을 분리할 수 있었다.

![](https://gunju-ko.github.io/assets/img/posts/toby-spring/aop/AOP-3.4.2.png)



### 그렇다면 AOP 란?

AOP는 Aspect Oriented Programming의 약자로 관점 지향 프로그래밍이라고 불린다.

&#x20;관점 지향은 쉽게 말해 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어서 보고 그 관점을 기준으로 각각 모듈화하겠다는 것이다.&#x20;

여기서 모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것을 말한다.&#x20;

AOP에서 각 관점을 기준으로 로직을 모듈화한다는 것은 코드들을 부분적으로 나누어서 모듈화하겠다는 의미다.&#x20;

이때, 소스 코드상에서 다른 부분에 계속 반복해서 쓰는 코드들을 발견할 수 있는 데 이것을 흩어진 관심사 (Crosscutting Concerns)라 부른다.&#x20;

#### AOP 주요 개념

* Aspect : 위에서 설명한 흩어진 관심사를 모듈화 한 것. 주로 부가기능을 모듈화함. Target : Aspect를 적용하는 곳 (클래스, 메서드 .. )&#x20;
* Advice : 실질적으로 어떤 일을 해야할 지에 대한 것, 실질적인 부가기능을 담은 구현체&#x20;
* JointPoint : Advice가 적용될 위치, 끼어들 수 있는 지점. 메서드 진입 지점, 생성자 호출 시점, 필드에서 값을 꺼내올 때 등 다양한 시점에 적용가능&#x20;
* PointCut : JointPoint의 상세한 스펙을 정의한 것. 'A란 메서드의 진입 시점에 호출할 것'과 같이 더욱 구체적으로 Advice가 실행될 지점을 정할 수 있음









ref:&#x20;

{% embed url="https://gunju-ko.github.io/toby-spring/2018/11/20/AOP.html" %}

{% embed url="https://www.oodesign.com/proxy-pattern" %}

{% embed url="http://velog.io/@dev_leewoooo/Dynamic-Proxy%EB%9E%80" %}
