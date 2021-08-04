# ch.8 고차 함수: 파라미터와 반환 겂으로 람다 사용

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

만약 더 다양한 조건들을 이용하여 만들고 싶다면 람다 다체를 받는것이 좋을것이다.

```kotlin
fun Array<SiteVisit>.averageDurationFor(predicate: (SiteVisit) -> Boolean): Double
        = log.filter(predicate).map(SiteVisit::duration).average()

fun main (){
    val average = log.averageDurationFor { it.os in setOf(OS.WINDOW, OS.ANDROID) }
    println(average)
}
```

코드 중복을 줄일 떄 함수 타입이 상당히 도움이 된다. 코드의 일부분을 복사해 붙여널고 싶은 경우가 있다면 그 코드르 람드로 만들면 중복을 제거할 수 있다.

