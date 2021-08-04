# ch2. 코틀린 기초

### 함수

```kotlin
fun max(a: Int, b: Int) : Int{
    return if (a>b) a else b
}
```

코틀린 함수의 기본 구조중 코틀린 if는\(값을 만들어 내지 못하는\) 문장이 아니고 **결과를 만드는 식**이라는 점이다. 이는 자바의 \(a &gt; b\) ? a : b 라는 식과 비슷하다.

### 식이 본문인 함수

위의 코드처럼 함수 본문이 하나의 식으로만 이루어져 있다면 간결하게 아래와 같이 간결하게 표현이 가능하다.

```kotlin
fun max(a: Int, b: Int) : Int = if (a>b) a else b

fun max(a: Int, b: Int) = if (a>b) a else b
```

* 블록이 본문인 함수 : 본문이 중괄호로 둘러싸인 함수
* 식이 본문인 함수 : 등호와 식으로 이뤄진 함수

### 변수

```kotlin
val a = "hello"        // 값이 정해져 있어서 타입 추론이 가능하므로 타입을 명시할 필요가 없음

val b : Int            // 초기값이 정해져 있지 않으므로 컴파일러가 타입추론을 할 수 없으므로 타입을 명시해야함
```

* val : 변경 불가능한 참조를 저장하는 변수, 자바의 final 변수에 해당
* var : 변경 가능한 참조

> 기본적으로 모든 변수를 val 키워드를 사용해 불변 변수로 선언하고, 나중에 꼭 필요할 때에만 var 로 바꿔서 사용하는것을 추천. 변경 분가능한 참조와 변경 불가능한 객체를 부수 효과가 없는 함수와 조합해 사용하면 코드가 함수형 코드에 가까워지기 때문.

var 변수는 변수의 값을 변경할 수 있지만 다른 타입으로 변경할 수 없다. 컴파일러는 변수 선언 시점의 초기화 식으로부터 변수의 타입을 추론하기 때문.

```kotlin
var answer = 42
answer = "hello"    // "Error: type mismatch"
```

### 문자열 템플릿

```kotlin
val name = "Ewan"
println("Hello $name")        //변수를 문자열 안에 $로 참조가능
println("Hello ${name}")    //중괄호로 감싸는 것이 오해를 피하기 쉽고 검색등 유용하니 습관을 들이자!
```

### class

Java

```java
public Person {
    private final String name;

    public Person (String name) {
        this.name = name;
    }

    public String getName(){
        return this.name;
    }
}
```

Kotlin

```kotlin
class Person(val name: String)
```

위의 코드는 같은 코드이며

코틀린의 기본 가시성은 public 이므로 이런 경우 변경자를 생략해도 된다.

### Property

자바에서는 필드와 접긎나를 한데 묶어 **프로퍼티\(property\)** 라고 한다. 코틀린은 프로퍼티를 언어 기본 능력으로 제공하며, 코틀린 프로퍼티는 자바의 필드롸 접근자 메소드를 완전히 대신한다.

```kotlin
cass Person {
    val name: String,        // 읽기 전용 프로퍼티로, 코틀린은 (비공개) 필드와 필드를 읽는 단순한 (공개) 게터를 만든다.
    var isMarried: Boolean    // 끌 수 있는 프로퍼티로, 코틀린은 (비공개) 필다, (공개) 게터  (공개) 세터를 만든다.
}
```

### 커스텀 접근자

아래 isSquare 에 접글할 때 마다 get\(\) 메소드가 일치하는지 실행하여 결과값을 돌려준다.

```kotlin
val isSquare: Boolean
    get() {
        return height == width
    }
```

### enum

```kotlin
enum class Color{
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

Kotlin 은 Java와 다르게 값만 열거하지 않고 프로퍼티도 존재할 수 있다.

```kotlin
enum class Color(
    val r: Int, val g: int, val b: Int
) {
    RED(255, 0, 0), 
    ORANGE(255, 0, 0), 
    YELLOW(255, 0, 0), 
    GREEN(255, 0, 0), 
    BLUE(255, 0, 0), 
    INDIGO(255, 0, 0), 
    VIOLET(255, 0, 0);    //마지막에는 반드시 세미콜론을 붙일것

    fun rgb() = (r * 256 + g) * 256 + b
}
```

### when

Java 의 switch 를 대체하는 구문

```kotlin
fun getMneonic(color: Color) =
    when(color) {
        Color.RED -> "RED"
        Color.ORANGE -> "ORANGE"
        Color.YELLOW -> "YELLOW"
        Color.BLUE -> "BLUE"
        Color.INDIGO, Color.VIOLET -> "INDIGO, VIOLET"    // java와 달리 break 쓸일없고, 두가지 조건또한 가능함
        else -> throw Exception("Not color")            // 매치되는 조건이 없으면 이 문장을 실행
    }
```

### while

```kotlin
while (조건) {
    /*...*/        // Java와 동일
}

do{
    /*...*/        // Java와 동일
} while (조건)
```

### for

```kotlin
for(i in 1..10){    // 범위값을 이용

}
```

### 예외처리

kotlin에서 try도 값처럼 사용할 수 있다.

```kotlin
val numver = try{
    Integer.parseInt(reader.readLine())
}catch (e: NumberFormatException){
    null    // 예외일때 null을 return
}
```

