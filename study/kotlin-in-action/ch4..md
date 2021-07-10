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

위의 클래스 Button의 showoff\(\) 메서드를 호출하면 에러를 내뿜는다.

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

| 변경자 | 클래스 맴버 | 최상위 선언 |
| :--- | :--- | :--- |
| public\(기본 가시성\) | 모든 곳에서 볼 수 있다. | 모든 곳에서 볼 수 있다. |
| internal | 같은 모듈 안에서만 볼 수 있다. | 같은 모듈 안에서만 볼 수 있다. |
| protected | 하위 클래스 안에서만 볼 수 있다. | \(최상위 선언에 적용할 수 없음\) |
| private | 같은 클래스 안에서만 볼 수 있다. | 같은 파일 안에서만 볼 수 있다. |

코틀린의 internal은 같은 모듈 내부에서만 볼 수 있는 것이다.

자바에서는 패키지가 같은 클래스를 선언하기만 하면 어떤 프로젝트의 외부에 있는 코드라도 패키지가 같은 클래스를 선언하기만 하면 어떤 프로젝트의 외부에 있는 코드라도 쉽게접근이 가능함. 그래서 모듈의 캡슐화가 깨짐.

> 코틀린은 가시성이 더 낮은 타입을 참조하지 못하게 한다. 어떤 클래스의 기반 타입 목록에 들어있는 타입이나 제네릭 클래스의 타입 파라미터에 들어있는 타입의 가시성은 그 클래스 자신의 가시성과 같거나 더 높아야 한다.

#### 1.4 내부 클래스와 중첩된 클래스: 기본적으로 중첩 클래스

클래스 안에 가른 클래스를 선언하면 좋은 이유 : 클래스를 캡슐화하거나 코드 정의를 그 코드를 사용하는 곳 가까이에 두고 싶을 때 유용하다.

| 클래스 B 안에 정의된 클래스 A | 자바에서는 | 코틀린에서는 |
| :--- | :--- | :--- |
| 중첩 클래스\(바깥쪽 클래스에 대한 참조를 저장하지 않음\) | static class A | class A |
| 내부 클래스\(바깥쪽 클래스에 대한 참조를 저장함\) | class A | inner class A |

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

* sealed class는 자동으로 open이다. \(당연히 하위 클래스들이 상속받고 제한하려는 의도이기 때문\)

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

