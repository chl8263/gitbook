# ch.5 람다로 프로그래밍

## 1. 람다 식과 맴버 참조

람다, 람다식\(lambda\) : 댜른 함수에 넘길 수 있는 작은 코드 조각.

### 람다식 문법

람다식은 항상 중괄호로 둘려쌓여 있고 '-&gt;' 를 통해 파라미터와 본문을 구분한다.

```text
 -- parameter --   -본문-
{x: Int, y: Int -> x + y}
```

람다식을 변수에 저장할 수도 있다.

```kotlin
val sum = {x: Int, y: Int -> x + y}

print(sum(1, 2))
```

어떤 Array 의가장 큰 값을 가지는 데이터를 뽑아내려면 다음과 같은 코틀린 라이브러리를 사용할 수 있다.

```kotlin
val people = arrayOf(Pair("a", 1), Pair("b", 2), Pair("c", 3))
people.maxByOrNull({it -> it.second})
```

람다 표현중 아래의 코드는 모두 같은 코드이며 마지막 코드가 제일 보기 깔끔하고 권장하는 방식.

```kotlin
// 명시적으로 인자를 표현하기
people.maxByOrNull({p: Pair<String, Int> -> p.first})

// 코틀린 추론으로 인해 it에 알아서 타입이 들어온다
people.maxByOrNull({it -> it.second})

// 마지막 인자는 () 밖에 중괄호로 뺄 수 있음 
people.maxByOrNull(){it -> it.second}

// ** 권장 **
// 인자가 하나라면 () 를 생각 가능함 , 
people.maxByOrNull {it -> it.second}

// ** 권장 **
// 코틀린 default 인자가 알아서 타입을 추론, 'it ->' 제거
people.maxByOrNull {it.second}
```

타입 추론이 안되는 경우 람다식 안에 파라메터를 명시햐야함

```kotlin
val sum = {x: Int, y: Int -> x + y}
```

### 1.4 현재 영역에 있는 변수에 접근

코틀린에서 람다에서 람다식 밖에 있는 파일널이 아닌 변수에 접근헐 수 있고, 그 변수를 변경할 수 있다.

람다 안에서 사용하는 외부 변수를 **람다가 포획한 변수** 라고 한다. 기본적으로 함수 안에 정의된 로컬 변수의 생명주기는 함수가 반환되면 끝난다.

하지만 어떤 함수가 자신의 로컬 변수를 포획한 람다를 반환하거나 다른 변수에 저장한다면 로컬 변수의 생명주기와 함수의 생명주기가 달라질 수 있다.

자바에서는 final 변수만 포획이 가능하다 하지만 코틀린에서는 아래의 코드처럼 변수를 람다가 포획할 수 있다.

```kotlin
var count = 0

var inc = {count ++}
```

어떻게 이런 현상이 가능할까??

아래 코드처럼 불현하는 객체를 하나 만들고 그 객체의 프로퍼티에 접근하여 마치 변수를 접근하도록 하는 것 처럼 보일 수 있게 한다.

하지만 아래 코드처럼 쓰지 않아도 위의 코드 처럼만 해도 된다.

```kotlin
class Ref<T>(var value: T)

>> val counter = Ref(0)
>> counter.values++
```

### **클로저**

람다를 실행 시점에 표현하는 데이터 구조는 함다에서 시작하는 모든 참조가 포함된 닫힌\(closed\) 객체 그래프를 람다 코드와 함께 저장해야 한다. 그런 데이터 구조를 클로져\(closer\) 라고 한다.

아래 코드는 test\(\) 함수가 새로운 함수를 반환한다. 이미 test 함수의 생성주기가 끝났음에도 불구하고 그 안에 있는 변수를 참조할 수 있다. 이는 외부 함수가 종료 되었음에도 해당 변수에 대한 참조를 가지고 있기 때문이다. 이를 클로져라고 한다.

```kotlin
fun test(): () -> Int {
    var count2 = 0

    return fun (): Int {
        return ++count2
    }
}

val a = test()
println(a())
println(a())

>> 1
>> 2
```

### 1.5 맴버 참조

함수를 값으로 비꾸는 방법은 `이중콜론` `::` 을 붙이는 것이다. 이를 맴버 참조 라고 한다.

맴버 참조는 프로퍼티나 메소드를 단 하나만 호출하는 함수 값을 만들어준다. `::` 는 이름과 `참조하려는 맴버(프로퍼티나 메소드)` 이름 사아에 위치한다.

```kotlin
-클래스- -맴버-
Person::age
```

맴버 참조는 람다와 같은 방식 이끼 때문에 아래는 모두 동등한 문법이다.

```kotlin
val persons = arrayListOf<Pair<String, Int>>(Pair("ewan", 1), Pair("ewan2", 2),Pair("ewan3", 3))

var result = persons.maxByOrNull { it-> it.second }
var result = persons.maxByOrNull { it.second }
var result = persons.maxByOrNull(Pair<String, Int>::second)
```

## 2. 컬렉션 함수형 API

### 필수적인 함수: filter와 map

`filter` 함수는 컬렉션에서 원하지 않는 를 제거한다. 하지만 `filter는 원소를 반환할 수 없다`. 원소를 반환 하려면 `map` 함수를 샤용해야 한다.

`map` 함수는 주어진 람다를 컬렉션의 각 원소에 적요한 결과를 모아서 새 컬렉션을 만든다.

```kotlin
val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.filter { it.age > 30 })

val people = listOf(Person("Alice", 29), Person("Bob", 31))
println(people.map { it.name })

println(people.filter { it.age > 30 }.map { Person::name })

val numbers = mapOf(0 to "zero", 1 to "one")
println(numbers.mapValues { it.value.toUpperCase() })
```

### 2.2 all, any, count, find: 컬렉션에 술어 적용

컬렉션에 대해 자주 수행하는 연산으로 컬렉션의 모든 원소가 어떤 조건을 만족하였는지판단하는 연산이 `all`, `any`이다.

`count` 함수는 조건을 만족하는 원소의 개수를 반환하며, `find`함수는 조건을 만족하는 `첫 번재` 원소를 반환한다.

아래 코드는 술어 함수를 하나 만들고 해당 술어에 컬렉션이 조건에 부합하는지 보는 코드이다.

```kotlin
val persons = arrayListOf<Pair<String, Int>>(Pair("ewan", 1), Pair("ewan2", 2),Pair("ewan3", 3))

data class Person(val name: String, val age: Int)

val canBeInClub27 = { p: Person -> p.age <= 27 }

val people = listOf(Person("Alice", 27), Person("Bob", 31))

println(people.all(canBeInClub27))
println(people.any(canBeInClub27))

>> false
>> true
```

### 2.4 flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리

flatMap 함수는 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에 적용하고 람다를 적용한 결과 얻어지는 여러리스트를 한 리스트로 한데 모은다.

```kotlin
fun main(args: Array<String>) {
    val strings = listOf("abc", "def")
    println(strings.flatMap { it.toList() })
}

>>[a, b, c, d, e, f]

fun main(args: Array<String>) {
    val books = listOf(Book("Thursday Next", listOf("Jasper Fforde")),
                       Book("Mort", listOf("Terry Pratchett")),
                       Book("Good Omens", listOf("Terry Pratchett",
                                                 "Neil Gaiman")))
    println(books.flatMap { it.authors }.toSet())
}

>> [Jasper Fforde, Terry Pratchett, Neil Gaiman]
```

컬렉션을 다루는 코드를 작성할 경우에는 원하는 바를 어떻게 일반적인 변환을 사용해 표현할 수 있는지 생각해보고 그런 변환을 제공하는 라이브러리 함수가 있는지 살펴보라.

## 3.지연 계산\(lazy\) 컬렉션 연산

> ! 싱글턴과 의존관계 주입

싱글턴 패턴은 소규모 프로젝트에는 적합 하지만 대규모 소프트웨어 시스템에서는 객체 선언이 항상 적합하지는 않는다.

이유는 객체 생성을 제어할 방법이 없고 생성자 파라미터를 지정할 수 없기 때문이다.

이런 상황에서는 의존관계 프레임워크를 사용하는 것이 좋다.

