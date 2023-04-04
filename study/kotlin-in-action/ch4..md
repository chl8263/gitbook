# ch4. 클래스, 객체, 인터페이스

### 1. 클래스 계층 정의

#### 1.1 코틀린 인터페이스

자바와 같이 **interface**로 표기한다.

class는 하나만 상속이 가능하고 interface는 여러개 구현이 가능함.

```kotlin
interface Clickable{
    fun click()        //추상 메소드 선언 : 구현하는 모든 클래스는 재정의 필요
}
```

함수 재정의는 생략이 가능한 java와는 달리 **ovveride** 변경자를 꼭 적어야 한다. **ovveride** 변경자는 실수로 상위 클래스의 메소드를 오버라이드하는 경우를 방지해준다.

```kotlin
class Button : Clickable {
    override fun click () = println("I was clicked")
}
```

Kotlin interface는 메소드 디폴트 구현이 가능하다.

```kotlin
interface Clickable{
    fun click()

    fun showoff() = println("I'm clickable")
}
```

그럼 만약 아래와 같이 동일한 디폴트 메서드를 가진 인터페이스를 두개 모두 상속받아 사용한다면?

```kotlin
interface Clickable{
    fun showoff() = println("I'm clickable")
}

interface Focusable{
    fun showoff() = println("I'm focusable")
}

class Button : Clickable, Focusable {
    ...
}
```

위의 클래스 Button의 showoff() 메서드를 호출하면 에러를 내뿜는다.

아래처럼 코틀린 컴파일러는 두 메소드를 아우르는 구현을 하위 클래스에 직접 구현하게 강제한다.

```kotlin
interface Clickable{
    fun showoff() = println("I'm clickable")
}

interface Focusable{
    fun showoff() = println("I'm focusable")
}

class Button : Clickable, Focusable {
    override fun showoff() {
        super<Clickable>.showoff()    // 상위 타입의 이름을 꺽쇠 괄호(<>) 사이에 넣어서 "super"를 지정하면
        super<Focusable>.showoff()    // 어떤 상위 타입의 맴버 메소드를 호출할지 지정할 수 있다.
    }
}
```

#### 1.2 open, final, abstract 변경자 : 기본적으로 final

Java 처럼 기본적으로 상속이 가능하면 의도하지 않은 하위 클래스가 기반 클래스에 대해 가졌던 가정이 기반 클래스를 변경함으로써 깨져버린 경우에 생긴다.

코틀린의 클래스와 매소드는 기본적으로 final 이다.

상속을 허용하려면 **open** 변경자를 붙여야 한다.

```kotlin
open class Button : AAA {

    fun disable() {}        // 이 함수는 final 이기 때문에 하위 클래스가 오버라이드 할 수 없음

    open fun able() {}        // 이 함수는 open 리기 때문에 하위 클래스가 오버라이드 할 수 있음

    override fun click() {} // 이 함수는 상위 클래스에서 재정의한 것, 기본적으로 open 이다

    final override fun showoff() {} // 오버라이드하는 메소드의 구현을 하위 클래스에서 오버라이드하지 못하게 금지하려면 앞에 final을 붙여라
}
```

코틀린의 추상클래스는 **abstract**변경자로 선언한다.

```kotlin
abstract class Animaled {            // 추상클래스, 이 클래스의 인스턴스를 만들 수 없음.

    abstract fun animate()            // 추상함수, 이 함수에는 구현이 없으며 하위 클래스는 무조건 재정의 해야함

    open fun stopAnimating() {}        // 비추상함수는 기본이 final이지만 원한다면 open할 수 있음

    fun animateTwice() {}            // 비추상함수의 기본은 final.
}
```

#### 1.3 가시성 변경자 : 기본적으로 공개

| 변경자            | 클래스 맴버              | 최상위 선언             |
| -------------- | ------------------- | ------------------ |
| public(기본 가시성) | 모든 곳에서 볼 수 있다.      | 모든 곳에서 볼 수 있다.     |
| internal       | 같은 모듈 안에서만 볼 수 있다.  | 같은 모듈 안에서만 볼 수 있다. |
| protected      | 하위 클래스 안에서만 볼 수 있다. | (최상위 선언에 적용할 수 없음) |
| private        | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |

코틀린의 internal은 같은 모듈 내부에서만 볼 수 있는 것이다.

자바에서는 패키지가 같은 클래스를 선언하기만 하면 어떤 프로젝트의 외부에 있는 코드라도 쉽게접근이 가능함. 그래서 모듈의 캡슐화가 깨짐.

> 코틀린은 가시성이 더 낮은 타입을 참조하지 못하게 한다. 어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 높아야 한다.

#### - 클래스 확장함수는 해당 클래스의 protected, private 에 접근할 수 없음

#### 1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

클래스 안에 가른 클래스를 선언하면 좋은 이유 : 클래스를 캡슐화하거나 코드 정의를 그 코드를 사용하는 곳 가까이에 두고 싶을 때 유용하다.

| 클래스 B 안에 정의된 클래스 A              | 자바에서는          | 코틀린에서는        |
| ------------------------------- | -------------- | ------------- |
| 중첩 클래스(바깥쪽 클래스에 대한 참조를 저장하지 않음) | static class A | class A       |
| 내부 클래스(바깥쪽 클래스에 대한 참조를 저장함)     | class A        | inner class A |

* 코틀린 에서 바깥쪽 클래스의 인스턴스를 가리키는 참조를 표기하는 방법

```kotlin
class Outer {
    inner class Inner {
        fun getOuterReference(): Outer = this@Outer
    }
}
```

#### 1.5 봉인된 클래스: 클래스 계층 정의 시 계층 확장 제한

중첩 클래스로 상위 클래스 계층에 속한 클래스의 수를 제한하고 싶은 경우 **Sealed**클래스를 사용하는 것이 좋다.

Rxpr 상위 클래스를 만들고 Num, Sum 을 표현하는 두 하위 클래스가 있고, 이를 이용하여 뭔가 하려고 할 때 아래와 같이 when 식을 이용하여 표현할 수 있다. 이때 Koltin 은 when 문에 else를 강제한다.

```kotlin
interface Expr

class Num(val value: Int) : Expr
class Sum(val lefValue: Int, val rightValue: Int) : Expr

fun eval (e: Expr): Int =
    when(e){
        is Num -> {
            ...
        }
        is Sum -> {
            ...
        }
        else -> {
            ...
        }
    }
```

하지만 **sealed** class를 사용하면 상위 클래스를 상속하는 하위 클래스 정의를 제한할 수 있기 때문에 when 문에 **else**를 붙일 필요가 없다.

```kotlin
sealed class Expr {
    class Num(val value: Int) : Expr()
    class Sum(val lefValue: Int, val rightValue: Int) : Expr()
}

fun eval (e: Expr): Int =
    when(e){
        is Expr.Num -> {
            ...
        }
        is Expr.Sum -> {
            ...
        }
    }
```

* sealed class는 자동으로 open이다. (당연히 하위 클래스들이 상속받고 제한하려는 의도이기 때문)

### 2 뻔하지 않은 생성자와 프로퍼티를 갖는 클래스 선언

#### 2.1 클래스 초기화: 주 생성자와 초기화 블록

클래스 이름 뒤에 오는 괄호로 둘러싸인 코드를 **주 생성자**라고 한다.

주 생성자는 생성자 파라미터를 지정하고 그 생성자 파라미터에 의해 초기화되는 프로퍼티를 정의하는 두 가지 목적에 쓰인다.

```kotlin
class User(val nickname: String)
```

위의 코드를 풀어서 쓰면 다음과 같이 쓸 수 있다.

```kotlin
class User constructor(_nickname: String) {    //constructor 는 주 생성자나 부 생성자를 정의할 때 사용됨
    val nickname: String

    init {    // 초기화 블록, 클래스의 객체가 만들어질 때(인스턴스화 될 때) 실행될 초기화 코드가 들어간다. 주 생성자와 함께 사용된다.

        nickname = _nickname    // _(밑줄은) 프로퍼티와 생성자 파라미터를 구분해준다.
    }
}
```

```kotlin
class User(val nickname: String,
           val isSubscribed: Boolean = true)    // 생성자 파라미터에 대한 디폴트 값을 제공한다.
```

클래스를 정의할 때 별도로 생성자를 정의하지 않으면 컴파일러가 자동으로 아무 일도 하지 않는 인자가 없는 디폴트 생성자를 만들어 준다.

```kotlin
open class Button
```

Buutton 의 생성자는 아무 인자도 받지 않지만, Button class 를 상속한 다른 class 들은 반드시 Button class의 생성자를 호출해야 한다.

이 규칙으로 인해 클래스는 생성자를 호출하고, 인터페이스는 생성자를 호출할 필요가 없어 클래스 및 인터페이스 목록에서 괄호를 보고 판단이 가능하다.

#### 2.2 부 생성자: 상위 클래스를 다른 방식으로 초기화

Kotlin 에서는 부 생성자로 class 초기화를 하는 방식이 존재한다.

**부 생성자로 초기화를 할 경우 반드시 주 생성자에게 생성을 위임하거나 상위 클래스에 생성을 위임 해야 한다.**

```kotlin
open class Button : View { // 상위 클래스 View 에 생성을 위임 하기 때문에 Button class 의 주 생성자가 필요 없음

    constructor(ctx: context) : super(ctx){     // 상위 클래스에 생성자를 위임함

    }

    constructor(ctx: context, attr: Attribute) : super(ctx, attr){

    }
}
```

```kotlin
open class Button(b: String) 

class A(a: String) : Button(a) {    // 주 생성자가 존재하고, 상위 클래스의 super 에게도 생성자를 주입함
    public var a: String = "asd"

    constructor(a: String, b: Int) : this(a){   // 주 생성자에게 생성자를 위임
        this.a = "12"
    }
}
```

#### 2.3 인터페이스에 선언된 프로퍼티 구현

interface 에 프로퍼티를 선언하고 그 interface를 구현하는 하위 클래스는 반드시 그 프로퍼티를 **override**해서 참조할 방법을 구현해야 한다.

```kotlin
interface AA {
    var a: String

    val b: String
        get() = "adsc"  // get() 메서드를 구현 했으므로 하위 클래스에서 구현이 필요 없음

    // val b : String = "hello" // interface 메서드를 직접 초기화할 수 없다.
}

class BB(override var a: String) : AA { // 인터페이스의 필드를 구현함

}
```

#### 2.4 게터와 세터에서 뒷받침하는 필드에 접근

* **val** 은 getter 만 생성되도 수정할 수 있다.
* **var** 는 getter, setter 둘 다 생성할 수 있다.

```kotlin
class BB() {
    var es: String = "hello"
        get() = "asc"
        set(v) {
            field = v
        }

    val isEqual
        get() = getIsEquals()

    private fun getIsEquals() = "a" == es

}
```

#### 2.5 접근자의 가시성 변경

setter 메서드에 private 접근자를 붙이면 해당 클래스 내부에서만 setter 를 호출할 수 있다.

```
var test: String = "hi!"
        private set
```

#### 3.1 모든 클래스가 정의해야 하는 메소드

자바와 마찬가지로 코틀린 클래스도 toString, equals, hashCode 등을 오버라이드할 수 있다.

* 문자열 표현 : toString()

Kotlin 기본적인 문자열 표현은 **BB@1d251891** 와 같다.

아래와 같이 toString()을 재정의 해서 좀더 가시성 있게 표현해야할 필요가 있다.

```kotlin
override fun toString(): String {
    return "Class(a = $a)"
}
```

* 객체의 동등성 : equals()

객체가 동등한지 판단하려면 Java 의 equals() 메소드를 이용하여 동등한지 판단할 것이다.

Kotlin 에서는 **==** 연산자가 이를 대신한다.

```kotlin
val client1 = Client("ewan1", 123)
val client2 = Client("ewan1", 123)

print(client1 == client2)
```

위의 코드의 결과는 false 를 출력한다. 두 객체가 동등하지 않는다는 것이다.

객체의 프로퍼티 값이 같을 때 동일한 객체라고 판단하기 위한 메서드를 추가하자.

```kotlin
override fun equals(other: Any?): Boolean {
    if(other == null || other !is A)    // Any(코틀린 최상위 클래스) 는 null이 될 수 있다. // 같은 객체 타입인지 검사
        return false
    return this.a == other.a
}
```

객체가 복잡한 경우 Equals 를 하나하나 정의할 수 없다. 그냥 equals 를 사용하면 안되는 이유는 **hashCode**가 다르기 때문이다. Java 에서 equal()가 true를 반환하는 두 객체는 반드시 hashCode 가 같아야 한다. 이제 hashCode 를 재정의 해보자.

* 해시 컨테이너: hashCode()

```kotlin
val processed = hashSetOf(Client("ewan", 1234))
print(processed.cointains(Client("ewan", 1234)))

>> false
```

위의 코드는 hashSet 을 하나 설정하고 갹체를 해쉬에 넣는다. 그리고 동일한 값을 가지는 객체를 contain 인자로 넣어 찾는다.

하지만 false 로 나오는 이유는 HashSet은 비용을 줄이기 위해 먼저 객체의 hashCode 비교하고 해시 코드가 같은 경우에만 실제 값을 비교 한다.

즉, hashSet 에서 hashcode 가 다를 때 equals가 반환하는 값은 판단 결과에 영향을 미치지 못한다는 것이다.

따라서 아래와 같이 메서드를 재정의 해야한다.

```kotlin
override fun hashCode(): Int = a.hashCode() *30 + 1234
```

#### 3.2 데이터 클래스: 모든 클래스가 정의해야 하는 메소드 자동 생성

윗 단원과 같이 koltin 에서 데이터를 다루는 클래스들은 반드시 **toString, equals, hashCode** 를 재정의 해야한다.

코틀린에서 이런 편의서을 고려하여 **data class** 를 제공한다. 이는 위의 모든 데이터를 다루는 클래스를 위한 메서드를 자동으로 만들어 준다.

* 인스턴스 간 비교를 위한 equals
* HashMap과 같은 해시 기반 컨테이너에서 키로 사용할 수 있는 hashCode
* 클래스 각 필드를 선언 순서대로 표시하는 문자열 표현읋 만들어 주는 toString

#### 데이터 클래스와 불변성: copy()

```kotlin
val lee = Client("ewan", 123)
print(lee.copy(postCode = 4000))    // 객체를 복사할 때 프로퍼티를 변경하여 새롭게 만들 수 있다.

>> Client("ewan", 4000)
```

#### 클래스 위임: by 키워드

대규모 갹체지향 시스템을 설계할 때 시스템을 취약하게 만드는 것은 보통 상속에서 발생한다.

상속을 하지 않아도 해당 클래스의 메서드 들을 마치 자신이 가진것 처럼 사용할 수 있게 만드는 대표적인 방법은 **Decorator**패턴이다.

하지만 테코레이터 패턴 구현에는 많은 코드 준비가 필요하다.

Kotlin은 **by**키워드를 사용히여 위임시킬 수 있다.

```kotlin
interface Vehicle { // 위임은 interface 만 가능하다
    fun go()
}

class Car() : Vehicle {
    override fun go() {
        println("Car gogo!")
    }
}

class Road(vehicle: Vehicle) : Vehicle by vehicle

public fun main(args: Array<String>){
    val vehicle = Car()
    val road = Road(vehicle)
    print(road.go())
}

>> Car gogo!
```

위의 코드와 같이 상속을 하지 않고도 마치 Vehicle 클래스 인것 처럼 사용할 수 있다.

&#x20;**위임은 interface 만 가능하다**&#x20;

#### 4. Object 키워드: 클래스 선언과 인스턴스 생성

코틀린에서 Object 키워드는 클래스를 생성함과 동시에 인스턴스를 생성한다.

**object**를 선언하는 상황은 다음과 같다.

* 객체선언 은 싱글턴을 정의하는 방법 중 하나다.
* 동반 객체(companion object) 는 인스턴스 메소드는 아니지만 어떤 클래스와 관련 있는 메솓와 팩토리 메소드를 담을 때 쓰인다. 동반 객체 메소드에 접근할 때는 동반 객체가 포함된 클래스의 이름을 사요할 수 있다.
* 객체 식은 자바의 **무명 내부 클래스(anonymous inner class)** 대신 씌인다.

#### 4.1 객체 선언: 싱글턴을 쉽게 만들기

Java 에서 private 키워드를 이용하여 생성자를 만들었던 방식과 달리 kotlin 에서는 언어 자체에서 싱글턴을 지원한다.

```kotlin
object Payroll{
    val allEmpliyee = ArrayList<String>()

    fun calculate(){
        //..
    }
}

Payroll.allEmpliyee // . 을 이용하여 프로퍼티, 메소드 접근가능
```

{% hint style="info" %}
### 싱글턴과 의존관계 주입

싱글턴 패턴은 소규모 프로젝트에는 적합 하지만 대규모 소프트웨어 시스템에서는 객체 선언이 항상 적합하지는 않는다.

이유는 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없기 때문이다.

이런 상황에서는 의존관계 프레임워크를 사용하는 것이 좋다.
{% endhint %}

코들린 클래스 안에는 정적인 맴버가 없다.

코틀린 언어는 Java의 Static 키워드를 지원 하지 않는다.

kotlin 의 클래스 내부에 private 에는 외부에서 접근할 수 없다.

하지만 **companion**접근자를 이용하면 클래스 내부에 접근할 수 있다.

정적 프로퍼티를 정의할 때나 아래와 같이 private 생성자를 이용하여 factory를 구현할 수 있다.

```kotlin
class User private constructor(val nickName: String) {
    companion object {
        fun newNormalUser(name: String) = User(name)
        fun newFacebookUser(accountId: String) = User(accountId)
    }
}
```

#### 4.3 동반 객체를 일반 객체처럼 사용

동반 객체는 클래스 안에 정의된 일반 객체이기 때문에 동반 객체에 이름을 붙이거나 동반 객체가 인터페이스를 상속하거나 동반객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

동반 객체에 이름을 정하고 그 이름으로 들어갈 수도, 바로 접근할 수도 있다.

```kotlin
class Person(val name: String) {
    companion object Loader{
        fun fromJSON(jsonText: String): Person {
            return Person(jsonText)
        }
    }
}

var person = Person.Loader.fromJSON("{name: ewan}")
var person2 = Person.fromJSON("{name: ewan}")
```

* 동반 객체는 인터페이스를 구현할 수도 있다.

```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

class Person(val name: String) {
    companion object Loader : JSONFactory<Person>{
        override fun fromJSON(jsonText: String): Person {
        }
    }
}
```

* 동반 객체의 확장함수

동반 객체의 확장 함수를 구현할 수도 있다. 동반객체에 이름을 반드시 넣어 주어야 하고 이름이 없다면 **Companion**으로 지정해 주면 된다.

```kotlin
class Person(val name: String) {
    companion object {  // 빈 동반 객체 선언, 이름을 따로 지정하지 않았음

    }
}

fun Person.Companion.fromJSON(jsonText: String): Person {   // 동반객체에 이름이 없기 때문에 Companion에 접근
    return Person(jsonText)
}

public fun main(args: Array<String>){
    Person.fromJSON("ewan")
}
```

4.4 객체식: 무명 내부 클래스를 다른 방식으로 작성

무명 객체를 표현할 때도 **object** 키워드를 작성한다.

```kotlin
interface JSONFactory<T> {
    fun fromJSON(jsonText: String): T
}

object : JSONFactory<String> {
    override fun fromJSON(jsonText: String): String {
        TODO("Not yet implemented")
    }
}
```

객체 서언과 달리 무명 객체는 싱글턴이 아니다. 객체 식이 쓰일 때마다 새로운 인스턴스가 생성된다.

### 요약

* 코틀린의 인퍼페이스는 자바 인터페이스와 바슷하지만 디폴트 구현을 포함할 수 있고, 프로퍼티도 포함할 수 있다.
* 모든 코틀린 선언은 기본적으로 final이며 public이다.
* 선언이 final이 되지 않게 만들려면 앞에 open 을 붙여야 한다.
* internal 선언은 같은 모듈 안에서만 볼 수 있다.
* 중첩 클래스는 기본적으로 내부 클래스가 아니다. 바깥쪽 클래스에 대한 참조를 중첩 클래스 안에 포함시키려면 inner키워드를 중첩 클래스 선언 앞에 붙여서 내부 클래스로 만들어야 한다.
* sealed 클래스를 상속하는 클래스를 정의 하려면 반드시 부모 클래스 정의 안에 중첩 클래스로 정의해야 한다.
* 초기화 블록과 부 생성자를 활용해 클래스 인스턴스를 더 유연하게 초기화할 수 있다.
* field 식별자를 통해 프로퍼티 접근자 안에서 프로퍼티의 데이터를 저장하느 데 쓰이는 뒷받침하는 필드를 참조할 수 있다.
* 데이터 클래스를 사용하면 컴파일러가 equals, hashcode, toString, copy 등 의 메소드를 자동으로 생성해 준다.
* 클래스 위임을 사용하면 위임 패턴을 구현할 때 필요한 수많은 성가진 준비 코드를 줄일 수 있다.
* 객체 선언을 사용햐면 코틀린 답게 싱클던 클래스를 정의할 수 있다.
* 동반 객체는 자바의 정적 메소드와 필드 정의를 대신한다.
* 동반 객체도 다른 갹체와 마찬가지로 인터페이스를 구현할 수 있다. 외부에서 동반 갹체에 대한 확장 함수와 프로퍼티를 정의할 수 있다.
