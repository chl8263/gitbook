# ch9. 제네릭스

## 1. 제네릭 타입 파라미터

제네릭스를 사용하면 `타입 파라미터`를 받는 타입을 정의할 수 있다. 예를들어 List 라는 타입이 있다면 그 안에 들어가는 원소의 타입을 알아야 할것이다.

타입 파라미터를 사용하면 해당 타입에 맞는 원소를 인스턴스화 할 수 있다.

예를들어 `Map<K, V>` 이라는 제네릭 클래스에 `Map<String, Person>` 처럼 구체적인 타입을 대입하면 인스턴스화할 수 있다.

### 1.1 제네릭 함수와 프로퍼티

제네릭 함수를 호출할 때는 반드시 구체적인 터입으로 타입 인자를 넘겨야 한다.

```kotlin
fun <T> List<T>.slice(indices: IntRange): List<T> // T가 넘어오는 타입 파라미터이다.
```

위 코드의 함수를 구체적인 리스트에 대해 호출할 떄 타입 인자를 명시적으로 지정할 수 있다. 하지만 대부분 컴파일러가 타입 인자를 추론할 수 있으므로 그럴 필요가 없다.

```kotlin
val letters = ('a'..'z').toList()

print(letters.slice<Char>(0..2))    // 타입 인자를 명시적으로 지정

print(letters.slice(0..2))    // 타입 인자를 컴파일러가 추론
```

제네릭 함수를 정의할 때와 마찬가지로 제네릭 확장 프로퍼티를 선언할 수 있다.

```kotlin
val <T> List<T>.penultimate: T
    get() = this[size-2]

fun main() {
    print(listOf("aaa", "bbb", "ccc", "ddd").penultimate)
}
```

{% hint style="info" %}
### 확장 프로퍼티만 제레닉하게 만들 수 있다.

일반 프로퍼티는 타입 파라미터를 가질 수 없다.
{% endhint %}

### 1.2 제네릭 클래스 선언

클래스에 `<>` 기호를 클래스 이름 뒤에 붙이면 클래스를 제네릭하게 만들 수 있다.

```kotlin
interace List<T> {
    operator fun get (index: Int): T
}

class StringList : List<String> {
    override fun get(index: Int): String = ....
}
```

### 1.3 타입 파라미터 제약

`타입 파라메터 제약`은 클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능이다.

예를들어 리스트의 속한 모든 원소의 합을 구하는 함수를 제네릭으로 구현 한다고 한다면 Int, Double 등과 같은 숫자 타입만 가능하게 해야하고 String 같은 다른 타입들은 불가하게 해야한다.

어떤 타입을 제네릭 타입의 파라미터에 대한 **상한**으로 지정한다면 그 제네릭 타입을 인스턴스화할 때 사용하는 타입 인자는 반드시 그 상한 타입이거나 그 상한 타입의 하위 타입 이여야 한다.

제약을 가하려면 이름 뒤에 `:` 콜론을 붙인다.

```kotlin
fun <T: Number> List<T>.sum(): T    // 제약을 숫자로 지정하여 숫자만 가능하게 한다.
```

제네릭을 비교 가능하게 하는 제약을 걸 수도 있다.

```kotlin
fun <T: Comaprable<T>> max(first: T, second: T): T {
    return if (first > second) first else second
}

println(max("aaaa", "bbb")) // String 은 알파벳 순으로 비교한다.
```

비교할 수 없는 타입을 호출하면 에러난다.

### 1.4타입 파라미터를 널이 될 수 없는 타입으로 한정

기본 제네릭의 타입은 `Any?` 로 된다.

```text
class Process<T> {
    fun process(value: T) {
        value?.hashcode()  // T 는 nullable
    }
}
```

따라서 null이 될 수 업세 만드려면 제네렉 타입을 직접 지정해야한다.

```text
class Process<T: Any> {
    fun process(value: T) {
        value.hashcode()  // T 는 nullable 이 아니다.
    }
}
```

 는 항상 null이 될 수 없는것을 보장한다.

## 2. 실행 시 제레닉스의 동작: 소거된 타입 파라미터와 실체화된 타입 파라미터

### 2.1 실행 시점의 제네릭: 타입 검사와 캐스트

코틀린 제네릭 타입 인자 정보는 런타임에 지워진다. 이는 제레닉 클래스 인스턴스가 그 인스턴스를 생성할 때 쓰인 타입 인자에 대한 정보를 유지하지 않는다는 뜻이다. 예를들어 실행 시점에 아래 두 리스트는 다른 타입의 값을 담을 수 있는 리스트지만 컴파일러는 그 둘을 그저 List으로만 기억한다.

```kotlin
val list1: List<String> = listOf("a", "b")
val list1: List<Int> = listOf(1, 2)
```

하지만 위의 두 List가 각 타입에 대해서만 값을 넣을 수있게 해주는데 이는 컴파일러가 타입 일자를 알고 올바른 타입의 값만 각 리스트에 넣을 수 있게 해주기 때문이다.

제네릭은 타입 인자를 따로 저장하지 않기 때문에 어떤 리스트가 문자열 리스트인지 정수 리스트인지 실행시점에 검사할 수 없다.

```text
if (value is List<String>) {...} // errror
```

is 검사에서 타이 인자로 지정한 타입을 검사할 수 없다.

이런 정보를 소거 함으로써 타입 정보의 크기가 줄어들어 전반적인 메모리 사용량이 줄어든다는 장점이 있다.

그렇다면 어떤 값이 집합이나 맵이 아니라 리스트인지 어떻게 알 수 있을까?

바로 `스타 프로젝션<*>` 을 사용하면 된다.

```kotlin
fun printSum(c: Collection<*>) {
    val intList: = c as? List<Int> ?: throw Exception("List is excepted")

    println(intList.sum())
}
```

```kotlin
printSum(listOf(1, 2, 3))       // 리스트이기 떄문에 통과한다.
printSum(setOf(1, 2, 3))        // set이기 때문에 as? 부분에서 예외가 발생한다.
printSum(listOf("a", "b", "c")) // 문자열 리스트이기 때문에 as?는 통과한다
                                // List 인줄은 알지만 그 안에 타입은 컴파일러가 모르기때문
                                // 하지만 intList.sum() 에서 에러가 날것.
```

하지만 아래코드처럼 `*` 이 아닌 `Int`처럼 타입을 명시한다면 코틀린 컴파일러는 타입을 잡아낼 수있다.

```kotlin
fun printSum(c: Collection<Int>) {
    val intList: = c as? List<Int> ?: throw Exception("List is excepted")

    println(intList.sum())
}
```

{% hint style="info" %}
#### 코틀린은 제레닉 함수의 본문에서 그 함수의 타입 인자를 가리킬 수 있는 특별한 기능을 제공하지 않는다.
{% endhint %}

### 2.2 실체화한 타입 파라미터를 사용한 함수 선언

아래 코드는 에러다. 코를린은 제레닉 타입을 저장하지 않기 때문에 타입을 알 수 없기 때문이다.

```kotlin
fun <T> isA(value: Any) = value is T
```

하지만 `inline`키워드를 통해 이것이 가능하게 할 수 있다. `inline`키워드는 컴파일 시점에 식을 모두 함수 본문으로 바꾸기 때문이다.

이때 `reified`라는 키워드 를 붙여 사용하면 컴파일 시점에 타입을 검사할 수 있다.

없으면 안된다.

```kotlin
inline fun <reified T> isA(value: Any) = value is T
```

{% hint style="danger" %}
#### inline 함수의 크기를 보고 사용하길 바란다. 함수가 커지면 실체화한 타입에 의존하지 않는 부분을 별도의 일반 함수로 뽑아내는 편이 낫다.
{% endhint %}

## 3. 변성: 제네릭과 하위 타입

변성: List과 List와 같이 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념

### 3.1 변셩이 있는 이유: 인자를 함수에 넘기기

List 타입에 List 타입을 넘기면 안전할까?

String 클래스는 Any를 확장하므로 Any 타입 값을 파라미터로 받는 함수에 String 함수를 넘겨도 절대 안전하다.

하지만 List 와 List 의 경우는 그렇지 않을 수 있을지도 모른다.

```kotlin
fun addAnswer(list: MutableList<Any>) {
    list.add(42)
}

val strings = mutableListOf("abc", "asd")
addAnswer(strings)

print(strings.maxBy { it.length })  // string 이기 때문에 에러 발생
```

위 코드와 같은 문제 때문에 과연 List 타입에 List 타입을 넘기면 안전할까에 대한 답은

**함수가 원소를 변경할때는 안전하지 않다.** 반대로 **함수가 원소를 변경하지 않는다면 결코 안전할 것이다.**

{% hint style="info" %}
#### 리스트의 변경 가능성에 따라 적절한 인터페이스를 선택하면 안전하지 못한 함수 호출을 막을 수 있다.
{% endhint %}

### 3.2 클래스, 타입, 하위 타입

올바른 제레닉 타입을 얻으려면 타입 파라미터를 구체적인 타입 인자로 바꿔줘야한다.

타입 사이의 관계를 논하기 위해 `하위 타입` 이라는 개념을 잘 알아야 한다. 어떤 타입 A의 값이 필요한 모든 장소에 어떤 타입 B의 값을 넣어도 아무 문제가 없다면 타입 B는 타입 A의 하위 타입이다.

코틀린의 상 하위 타입 구조는 아래와 같다.

```text
Int  --- O ---> Number
Int  --- O ---> int
Int  --- X ---> String

Int  --- O ---> Int?
Int? --- X ---> Int
```

{% hint style="danger" %}
#### 널이 될 수 없는 타입은 널이 될 수 있는 타입의 하위 타입이다. 하지만 두 타입 모두 같은 클래스에 해당한다.

#### 항상 널이 될 수 없는 타입의 값을 널이 될 수있는 타입의 변수에 저장할 수 있지만 거꾸로 널이 될 수 있는 타입의 값을 널이 될 수 없는 타입의 값에 저장할 수 없다.
{% endhint %}

### 3.3 공변성: 하위 타입 관계를 유지

A가 B의 하위 타입이면 Producer는 Producer의 타위 타입이다. 여기서 Producer는 `공변적`이다.

또 클래스나 인터페이스를 `공변적`이라고 한다.

예를들어 Cat이 Animal의 하위 타입 이기때문에 Producer 은 Producer의 하위 타입이다.

코틀린에서 제네릭 클래스가 타입 파라미터에 대해 공변적임을 표시하려면 타입 파라미터 이름 앞에 `out`을 넣어야 한다.

```kotlin
interfacce Producer<out T> { // 클래스 T에 대해 공변적이라고 선언한다.
    fun produce(): T
}
```

```text
open class Animal {
    fun feed() { ... }
}

class Cat : Animal {
    ...
}

class Herd<T: Animal> {
    val size: Int
        get() = ...
        ....
}

fun feedAll(animals: Herd<Animal>){
    ...
}

fun takeCare(cats: Herd<Cat>){  // error
    ...
}
```

위의 코드에서 `takeCare` 함수를 실행시키면, 타입 에러가 발생한다. `Herd 클래스의 T 타입에 아무런 변성도 지정하지 않았기 때문`이다.

타입 캐스트로도 이를 해결할 수 있지만 이는 코드를 장황하게하고 실수를 만든다.

그래서 Herd를 공변적인 클래스로 만들고 호출 코드를 적절히 바꿀 수있다.

```text
open class Animal {
    fun feed() { ... }
}

class Cat : Animal {
    ...
}

class Herd<out T: Animal> { // T 는 이제 공변적이다.
    val size: Int
        get() = ...
        ....
}

fun feedAll(animals: Herd<Animal>){
    ...
}

fun takeCare(cats: Herd<Cat>){  // 컴파일에러가 나지않는다.
    ...
}
```

**타입 안전성을 보장하기 위해 공변적 파라미터는 항상 `아웃` 위치만 있어야 한다.** 이는 클래스가 T타입의 값을 생산할 수 있지만 T 타입의 값을 소비할 수는 없다는 뜻이다.

클래스 맴버를 선언할 때 타입 파라미터를 사용할 수 있는 지점은 모두 `인`과 `아웃`위치로 나뉜다.

* T 가 함수의 반환 타입에 쓰인다면 T는 `아웃` 위치에 있다. 그 함수는 T 타입의 값을 `생산`한다.
* T 가 함수의 파라미터 타입에 쓰인다면 T는 `인` 위치에 있다. 그 함수는 T 타입의 값을 `소비`한다.

```kotlin
interface Transformer<T> {
    fun transform(t: T):  T
                  인 위치  아웃위치
}
```

* out 키워드는 T 앞에 out 키워드를 붙이면 클래스 안에서 T를 사용하는 메소드가 `아웃 위치에서만 T를 사용`하게 허용하고 `인 위치에서는 T를 사용하게못하게` 한다.

{% hint style="info" %}
#### 타입 파라미터 T 에 붙은 out 키워드는 다음 두 가지를 함께 의미한다.

* 공변성 : 하위 타입 관계가 유지된다.
* 사용 제한 : T를 아웃 위치에서만 사용할 수 있다.
{% endhint %}

예를들어 List 인터페이스를 보자

```kotlin
public interface List<out E> : Collection<E> {

    ...

    public operator fun get(index: Int): E  // 읽기 전용 메소드로 T를 반환하는 메소드만 정의한다. 따라서 T는 항상 '아웃'의 위치에만 쓰인다.
```

반대로 MutableList는 공변적일 수 없다. T 를 인자로 받아서 그 타입의 값을 반환하는 메소드가 있다.

```kotlin
public interface MutableList<E> : List<E>, MutableCollection<E> {

    override fun add(element: E): Boolean   // 인의 위치에 쓰인다.

    ...
```

