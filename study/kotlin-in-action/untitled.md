# ch.8 고차 함수: 파라미터와 반환 값으로 람다 사용

## 1. 고차 함수 정의

고차 함수는 `다른 함수를 인자로 받거나` `함수를 반환`하는 함수다.

예를들어 표준 라이브러리인 filter는 술어 함수를 인자로 받으므로 고차 함수다.

```kotlin
list.filter { x > 0 }
```

고차함수를 정의하려면 `함수 타입`에 대해 먼저 알아야 한다.

### 1.1 함수 타입

코틀린 타입 추론으로 인해 변수 타입을 지정하지 않아도 람다를 변수에 대합할 수 있다.

```kotlin
val sum = { x: Int, y: Int -> x + y }
val action = { print(12) }
```

위의 코드의 경우 컴파일러가 함수타입을 추론한다.

각 변수에 구체적인 함수 타입 선언을 하면 아래와 같다.

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
val action: () -> Unit = { print(12) }
```

함수 타입을 정의하려면 함수 파라미터의 타입을 괄호 안에 넣고, 그 뒤에 화살표를 추가한 다음,함수의 반환 타입을 지정하면 된다.

함수 타입에서도 반환 타입을 널이 될 수 있는 타입으로 지정 가능하다.

```kotlin
val sum: (Int, Int) -> Int? = { x: Int, y: Int -> null }
```

함수 자체가 널이 될 수 있다면, 전체를 괄호로 감싸면 된다.

```kotlin
val sum: ((Int, Int) -> Int)? = null
```

### 1.2 인자로 받은 함수 호출

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(1, 2)
    println(result)
}

twoAndThree { a, b -> a + b }
```

### 1.3 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터

함수 타입 파라미터를 디폴트로 지정이 가능하다.

```kotlin
fun <T> Collection<T>.JoinToString(
    ...
    ...
    transform: (T) -> String = { it.toString() }    // it으로 지정가능
){
    ...
}
```

함수 타입의 디폴트가 널이 될 수 있다면 `엘비스`연산자로 안전한 호출을 해주는 것이 좋다.

```kotlin
fun <T> Collection<T>.JoinToString(
    ...
    ...
    transform: ((T) -> String)? = null    // null을 반환
){

    for .. {
        val str = transform(it) ?: it.toString()
    }
}
```

### 1.5 함수를 함수에서 반환

아래 코드는 함수를 반환 한다.

```kotlin
fun test(component: (Int, Int) -> Int): (Int) -> Boolean {
    val a = component.invoke(1,2)    // 인자로 받은 함수를 실행
    return { it -> it > 12 }        // 람다를 반환
}
```

### 1.6 람다를 활용한 중복 제거

함수 타입과 함다 식은 재활용하기 좋은 코드를 만들 떄 쓸 수 있는 좋은 방식이다.

웹사이트 방문 기록을 분석하는 예를 살펴보자

다음과 같은 데이터가 있다.

```kotlin
data class SiteVisit (
    val path: String,
    val duration: Double,
    val os: OS
        )

enum class OS {
    WINDOW, LINUX, MAC, IOS, ANDROID
}

val log = arrayOf(
    SiteVisit("/", 34.0, OS.WINDOW),
    SiteVisit("/", 18.0, OS.WINDOW),
    SiteVisit("/", 19.0, OS.LINUX),
    SiteVisit("/", 123.0, OS.MAC),
    SiteVisit("/", 92.0, OS.IOS),
    SiteVisit("/", 9.0, OS.ANDROID),
    SiteVisit("/", 12.0, OS.WINDOW),
)
```

이떄 평균 방문 시간을 호출하고 싶으면 아래와 같이 하면 된다.

```kotlin
val average = log.map { it.duration }.average()
println(average)
```

Window 사용자만 뽑아내려면 filter를 사용하면 된다.

```kotlin
val average = log.filter { it.os == OS.WINDOW }.map { it.duration }.average()
```

중복을 피하기 위해 os 를 인자로 받는 확장함수겸 고차함수를 만들자

```kotlin
fun Array<SiteVisit>.averageDurationFor(os: OS): Double
        = log.filter { it.os == os }.map(SiteVisit::duration).average()


fun main (){
    val average = log.averageDurationFor(OS.LINUX)
    println(average)
}
```

만약 더 다양한 조건들을 이용하여 만들고 싶다면 람다 자체를 받는것이 좋을것이다.

```kotlin
fun Array<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean): Double
        = log.filter(predicate).map(SiteVisit::duration).average()

fun main (){
    val average = log.averageDurationFor { it.os in setOf(OS.WINDOW, OS.ANDROID) }
    println(average)
}
```

코드 중복을 줄일 떄 함수 타입이 상당히 도움이 된다. 코드의 일부분을 복사해 붙여널고 싶은 경우가 있다면 그 코드르 람드로 만들면 중복을 제거할 수 있다.

## 2. 인라인 함수: 람다의 부가비용 없애기

`inline`변경자를 어떤 함수에 붙이면 컴파일러는 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해준다.

### 2.1 인라이닝이 작동하는 방식

어떤 함수를 inline으로 선어하면 그 함수의 본문이 인라인이 된다. 즉, 함수를 호출하는 코드를 함수를 호출하는 바이트코드 대신에 함수 본문을 번역한 바이트 코드로 컴파일한다는 뜻이다.

아래 코드의 `action` 함수를 실행시킬 때 `synchronized` 함수와 같이 본문에 포함이 된다.

```kotlin
inline fun <T> synchronized(lock: Lock, action: () -> T): T {
    lock.lock()

    try{
        return action()
    }

    finally {
        lock.unlock()
    }
}

fun foo(l: Lock) {
    println("before lock")

    synchronized(1) {
        println("synchronized")
    }

    println("after lock")
}
```

즉, 위의 `foo` 함수를 실행 시키면 아래와 같이 컴파일 된다.

```kotlin
fun foo(l: Lock) {
    println("before lock")

    lock.lock()

    try{
        println("synchronized")
    }

    finally {
        lock.unlock()
    }

    println("after lock")
}
```

### 2.2 인라인 함수의 한계

인라이닝을 하는 방식으로 인해 람다를 사용하는 모든 함수를 인라이닝할 수는 없다.

함수가 인라이닝될 때 그 함수에 인자로 전달된 람다 식의 본문은 결과 코드에 직접 들어갈 수 있다. 하지만 이렇게 람다가 본문에 직접 펼쳐지기 때문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정될 수밖에 없다.

함수 본문에서 파라미터로 받은 람다를 호출하면 그 호출을 쉽게 람다 본문으로 바꿀 수 있다.

**하지만 파라미터로 받은 람다를 다른 변수로 저장 했다가 나중에 사용한다면 람다를 표현하는 객체가 어딘가는 존재해야 하기 때문에 람다를 인라이닝할 수 없다.**

둘 이상의 람다를 인자로 받는 함수에서 일부 람다반 인라이닝하고 싶을 경우 `noninline`키워드를 붙이면 된다.

```kotlin
inline fun <T> synchronized(action: () -> T, noninline transform: () -> T): T {

}
```

### 2.3 컬렉션 연산 인라이닝

코틀린 표준 라이브러리의 컬렉션 함수는 대부분 람다를 인자로 받는다. 표준 라이브러리 함수를 사용하지 않고 직접 이런 연산을 구현한다면 더 효율적이지 않을까?

```kotlin
people.filter { it.age > 12 }
```

{% hint style="info" %}
코틀린의 filter 함수는 인라인 함수다. 따라서 filter 함수의 바이트 코드는 그 함수에 전달된 람다 본문의 바이트 코드와 함꼐 filter를 호출한 위치에 들어간다
{% endhint %}

그렇다면 filter와 map같이 연쇄해서 사용하는 경우는 어떨까?

```kotlin
people.filter { it.age > 12 }.map(Person::age)
```

위코드의 filter, map 함수는 인라인 함수다 따라서 추가적인 객체나 클래스 생성은 없다. 하지만 이 코드는 리스트를 걸러낸 결과를 저장하는 중간 리스트를 만든다. 

처리할 원소가 많아지면 `sequence` 를 통해 리스트 대신 시퀀스를 사용하기 때문에 중간 리스트로 인한 부가 비용은 줄어든다. 이때 각 중간 시퀀스를 람다를 필드에 저장아하는 객체로 표현되며, 최종 연산은 중간 시퀀스에 있는 여러 람다를 연쇄 호출한다. 따라서 시퀀스는 람다를 인라인하지 않는다.

{% hint style="danger" %}
### Sequence 는 람다를 인라인하지 않는다.

지연계산을 통해 성능을 향상시키려는 이유로 모든 컬렉션 연산에 sequence 를 붙여서는 안된다. 시퀀스 연산은 람다를 인라인 하지 않기 때문에 크기가 작은 컬렉션은 오히려 일반 컬렉션연산이 더 성늘이 좋을 수 있다.

시퀀스를 통해 성능을 향상시킬 수 있는 경우는 컬렉션 크기가 큰 경우 뿐이다.
{% endhint %}

### 2.4 함수를 인라인으로 선언해야 하는 이유

inline 키워드를 사용해도 람다를 인자로 받는 함수만 성능이 좋아질 가능성이 높다.

일반 함수 호출의 경우 JVM이 이미 강력하게 인라인을 지원하는데 바이트 코드를 실제 기계어 코드로 변역하는 과정\(JIT\) 에서 일어난다.

{% hint style="info" %}
인라이닝하는 함수가 큰 경우 함수의 본문에 해당하는 바이트코드를 모든 호출 저점에 복사해 넣고 나면 바이트코드가 전체적으로 아주 커질 수 있다.

코틀린 라이브러리가 제공하는 inline함수는 모두 코드가 작다.
{% endhint %}

## 3. 고차 함수 안에서 흐름 제어

### 3.1 람다 안의 return문: 람다를 둘러싼 함수로부터 반환

아래 코드의 for 문안에서 조건에 맞는다면, 함수를 return 시키는 것을 볼 수 있다.

```kotlin
fun something() {
    for (person in Persons) {
        if(person.name == "ewan") {
            println("")
            return
        }
    }
}
```

forEach 로 바꾸어 쓴다면?

```kotlin
fun something() {
    Persons.forEach {
        if(person.name == "ewan") {
            println("")
            return
        }
    }
}
```

위의 람다 안에서 return을 하고 있다. 람다식 이 함수인데 람다를 종료하지 않고 첫번째 코드처럼 자신을 감싸고 있는 블록보다 더 바깥에 있는 다른 블록을 반환하게 한다.

이런 return을 `non-local return` 이라고 부른다.

이런 경우는 `람다를 인자로 받는 함수가 inline`일 경우에만 가능하다.

### 3.2 람다로부터 반환: 레이블을 사용한 return

함수나 람다에 레이블을 지정하여 return을 자정한 레이블에 하게 할 수 있다.

```kotlin
fun something() {
    Persons.forEach ewan@{    // label 지정
        if(person.name == "ewan") {
            println("")
            return@ewan    // 람다만 return 하게 된다.
        }
    }
}
```

람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 붙여도 된다.

```kotlin
fun something() {
    Persons.forEach {
        if(person.name == "ewan") {
            println("")
            return@forEach
        }
    }
}
```

### 8.3 무명 함수: 기본적으로 로컬 return

람다의 `nonlocal return` 이나 이를 방지하기 위해 `label`을 사용할 시에 코드가 복잡해지고 장황해질 수 있다.

이를 해결하기 위해 `무명객체`를 사용할 수 있다.

```kotlin
Persons.forEach(
    fun (person): Boolean {    // 파라미터에 Persons의 원소가 들어간다.
        if(person.name == "ewan") {
            println("")
            return
        }
    }
)
```

무명객체를 사용하면 가장 가까운 함수를 return 하기 때문에 무명함수 이외의 다른 것도 반환하게 하지 못한다.

이를 잘 활용해라.

## 요약

* 함수 타입을 사용해 함수에 대한 참조를 담는 변수나 파라미터나 반환 값을 만들 수 있다.
* 고차 함수는 다름 함수를 인자로 받거나 함수를 반환한다. 함수의 파라미터 타입이나 반환 타입으로 함수 타입을 사용하면 고차 함수를 선언할 수 있다.
* 인라인 함수를 컴파일할 때 컴파일러는 그 함수의 본문과 그 함수에게 전달죈 람다의 본문을 컴파일한 바이트코드를 모든 함수 호출 지점에 삽입해준다.
* 고차함수를 사용하면 컴포넌트를 이루는 각 부분의 코드를 더 잘 재사용할 수 있다.
* 인라인 함수에서는 람다 안에 있는 return문이 바깥쪽 함수를 반환시키는 넌로컬 return을 사용할 수 있다.
* 무명 함수는 람다 식을 대신할 수 있으면 return 식을 처리하는 규칙이 일반 람다식과는 다르다. 본문 여러곳에서 return해야 하는 코드 블록을 만들어야 한다면 람다 대신 무명 함수를 쓸 수 있다.

