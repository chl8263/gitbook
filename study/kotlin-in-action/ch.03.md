# ch3. 함수 정의와 호출

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    seperator: String,
    prefix: String,
    postfix: String
): String {
    ......
}

joinToString(listOf(0, 1, 2), ";", ";", ",")

joinToString(collection = listOf(0, 1, 2), seperator = ";", prefix = ";", postfix = ",")    // 명시적으로 이름에 인자를 붙일 수 있음
```

### 디폴트 파라미터값

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    seperator: String = ";",
    prefix: String = ",",
    postfix: String = ","
): String {
    ......
}

joinToString(listOf(0, 1, 2))    // 파라메터가 선언이 되어있으면 생략이 가능하다.
```

### 정적인 유틸리티 클래스 없애기: 최상위 함수와 프로퍼티

코틀린에서는 함수가 꼭 class 안에 존재해야할 필요는 없다.

어떤 함수를 String 패키지에 넣으려면 예를들어 Join.Kt 파일을 만들고

```kotlin
package strings;

fun joinToString(...): String {...}
```

위와 같이 package 명을 적고 그냥 함수를 작성하기만 하면 끝이다.

코틀린은 컴파일할 때 아래와 같이 파일명을 기준으로 class 파일을 만들어준다.

```java
package strings;

public class JoinKt{
    public static String joinToString (...) {...}
}
```

따라서 자바에서 joinToString을 호출 하려면 아래와 같이 할 수 있다.

```kotlin
import strings.JoinKt;
...
JoinKt.joinToString(list, ";", ",", ":");
```

### 메소드를 다른 클래스에 추가: 확장 함수와 확장 프로퍼티

#### 확장 함수는 어떤 클래스의 맴버 메소드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수다.

확장 함수를 만들려면 추가하려는 함수 이름 앞에 그 함수가 확장할 클래스의 이름을 덧붙이기만 하면 된다. 클래스 이름을 **수신 객체 타입**이라 부르며, 확장 함수가 호툴되는 대상이 되는 값을 **수신 객체**라고 한다.

```kotlin
  수신객체 타입               수신객체   수신객체
      |                         |        |
     \ /                       \ /      \ /
fun String.lastChar(): Char = this.get(this.length-1)

println("Hello".lastChar())    // 위의 함수를 호출하는 구문은 일반 클래스 맴버를 호출하는 구문과 같다.
```

확장함수를 사용하고 import를 해줘야 그 함수를 사용할 수 있음

```kotlin
import strings.lastchar

val c = "Hello".lastchar
```

### 확장 프로퍼티

확장 함수와 같이 확장 프로퍼티 또한 적용할 수 있다.

```kotlin
val String,lastChar: Char        // get만 만들때 val
    get() = this.get(length-1)

val String,lastChar: Char        // set도 만들고 싶을때는 var
    get() = this.get(length-1)
    set(value: String) {
        this.setChar(length-1, value)
    }
```

#### 값의 쌍 다루기

map 을 만들려면 **mapOf** 함수를 사용한다.

```kotlin
val map = mapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
```

여기서 나오는 **to**는 코틀린 키워드가 아니다. 이 코드는 **중위 호출**이라는 특별한 방식으로 to라는 일반 메소드를 호출한 것이다.

다음 두 호출은 동일하다

```text
to("one")    <--- "to" 메소드를 일반적인 방식으로 호출
to "one"     <--- "to" 메소드를 중위 호출 방식으로 호출함
```

인자가 하나뿐인 일반 메소드나 인자가 하나뿐인 확장 함수에 중위 호출을 사용할 수있다. 함수를 중위 호출에 사용하게 허용하고 싶으면 **infix** 변경자를 함수 선언 앞에 추가해야 한다.

다음은 to 함수를의 정의를 간략하게 줄인 코드다.

```kotlin
infix fun Any.to(otherL Any) = Pair(this, other)
```

이를 이용하여 코틀린에서 **구조 분해**를 사용할 수 있다.

```kotlin
val (number, name) = 1 to "one"
```

