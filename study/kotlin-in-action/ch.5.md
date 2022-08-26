# ch5. 람다로 프로그래밍

## 1. 람다 식과 맴버 참조

람다, 람다식(lambda) : 댜른 함수에 넘길 수 있는 작은 코드 조각.

### 람다식 문법

람다식은 항상 중괄호로 둘려쌓여 있고 '->' 를 통해 파라미터와 본문을 구분한다.

```
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

람다를 실행 시점에 표현하는 데이터 구조는 함다에서 시작하는 모든 참조가 포함된 닫힌(closed) 객체 그래프를 람다 코드와 함께 저장해야 한다. 그런 데이터 구조를 클로져(closer) 라고 한다.

아래 코드는 test() 함수가 새로운 함수를 반환한다. 이미 test 함수의 생성주기가 끝났음에도 불구하고 그 안에 있는 변수를 참조할 수 있다. 이는 외부 함수가 종료 되었음에도 해당 변수에 대한 참조를 가지고 있기 때문이다. 이를 클로져라고 한다.

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

## 3.지연 계산(lazy) 컬렉션 연산

map 과 filter같은 함수는 컬렉션을 즉시 생성한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 게산 중간 결과를 새로운 컬렉션에 임시로 담는다는 말이다. `시퀀스` 를 사용하면 중간 임시 컬렉션을 사용하지 않고도 컬렉션 연산을 연쇄할 수 있다.

```kotlin
people.map(Person::name).filter {it.startWith("A") }
```

위의 코드는 연쇄 호출이 리스트를 2번 만든다는 뜻이고, 이 연쇄가 100번이면 굉장히 비효율적임.

이를 더 효율적으로 만들기 위한 것이 `시퀀스`다.

```kotlin
data class Person(val name: String, val age: Int)
val people = listOf(Person("Alice", 27), Person("Bob", 31))

people.asSequence()
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()    // 컬렉션으로 변환
```

위의 코드는 중간 결과를 따로 저장하거나 생성하지 않기 때문에 성능이 눈에 띄게 좋아진다.

시퀀스에 대한 연산을 `지연` 계산하기 때문에 정말 계산을 실행하게 만들려면 최종 시퀀스의 원소를 하나씩 이터레이션하거나 최종 시퀀스를 리스트로 변환해야 한다.

### 3.1 시퀀스 연산 실행: 중간 연산과 최종 연산

시퀀스에 대한 연산은 `중간 연산`과 `최종 연산` 으로 나뉜다. 중간 연산은 `다른 시퀀스를 반환`한다. `최종 연산`은 결과를 반환한다.

`중간 연산`은 항상 지연 계산된다. 최종 연산이 없는 (컬렉션으로 변환) 경우 아무 내용도 출력되지 않는다.

```kotlin
people.map(Person::name)
    .filter { it.startsWith("A") }
```

위의 코드처럼 시퀀스가 아닌 직접 연산을 할 경우 map함수에 대해 먼저 시퀀스를 얻고 그 시퀀스에 대해 다시 filter를 수행할 것이다.&#x20;

하지만 시퀀스의 경우 모든 연산은 각 원소에 대해 순차적으로 적용된다. 즉 첫번째 원소가 처리되고, 다시 두 번째 원소가 처리되며, 이런 처리가 모든 원소에 대해 적용된다.

아래의 그림처럼 시퀀스는 컬렉션을 만들지 않고 시퀀스로 map연산에서 이미 find 를 찾았으면 더이상 뒤의 연산을 하지 않아도 되기때문에 효율적이다.

![](<../../.gitbook/assets/image (18).png>)

filter, map 또한 순서에 따라 연산 횟수가 다르다

![](<../../.gitbook/assets/image (19) (1).png>)

## 4.자바 함수형 인터페이스 활용

코틀린은 자바와 호환이 가능함.

```kotlin
button.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View v){
      //...
    }
});
```

위와 같이 Button에 Click 발생시 그 이벤트를 캐치할 수 있는 ClickListener를 등록 할 수 있다. 그리고 이때 위와 같이 자바는 무명클래스의 인스턴스를 만들어야 함.

코틀린에서는 위의 무명클래 인스턴스 대신 람다를 넘겨줄 수 있다.

```kotlin
button.setOnClickListener{view -> ...}
```

{% hint style="info" %}
이런 코드가 작동하는 이유는 OnClickListner에 추상 메소드가 단 하나만 있기 때문. 그런 인터페이스를 함수형 인터페이스 또는 SAM 인터페이스라고 한다. (SAM : Single abstract method) 자바는 위와 같이 함수형 인터페이스를 활용하는 메소드가 많기 때문에 코틀린에서 함수형 인터페이스를 인자로 취하는 자바 메소드를 호출할 때 람다를 넘길 수 있게 해준다.
{% endhint %}

### 4.1 자바 메소드에 람다를 인자로 전달

다음 자바 메소드를 코틀린에서 람다를 전달할 수 있다.

```kotlin
void postphoneCoputation(int delay, Runable computation);
```

위에 메소드는 int완 Runable 2개의 인자를 가지는데. 여기서 Runable 부분을 람다를 사용하여 전달을 하면 `컴파일러는 자동으로 람다를 Runable로 변환해준다.`

```kotlin
postphoneComputation(1000) { println(42) }
```

위의 람다 코드와 다르게 Runable을 명시적으로 만들 수 있다.

```kotlin
postphoneComputation(1000, object:Runable{
    override fun run(){
        println(42)
    }
})
```

{% hint style="info" %}
객체를 명시적으로 선언하는 경우 메소드를 호출할 때마다 새로운 객체가 생성된다. - **매번 인스턴스를 생성함**

람다는 정의가 들어있는 함수의 변수에 접근하지 않는 람다에 대응하는 무명객체를 메소드를 호출할 때마다 반복 사용한다. - **매번 인스턴스를 재사용함**
{% endhint %}

{% hint style="danger" %}
람다를 사용할 경우에도 반드시 반복사용하는 것은 아니다.&#x20;

람다가 주변 영역의 변수를 사용하게 된다면 컴파일러는 주변 영역 변수를 사용하는 새로운 인스턴스를 생성하게 된다.

```kotlin
fun handleComputation(id : string){
    postphoneComputation(1000) { println(id) }
}
```
{% endhint %}

{% hint style="danger" %}
### 람다와 리스너 등록/해제하기

람다에는 무명 객와 달리 인스턴스 자신을 가리키는 this 가 없다.

따라서 람다를 변환한 무명 클래스의 인스턴스를 참조할 방법이 없다. 컴파일러 입장에서 보면 람다는 코드블록일 뿐이고, 객체가 아니므로 객체처럼 람다를 참조할 수는 없다.

람다 안에서 this는 그 람다를 둘러싼 클래스의 인스턴스를 리킨다.
{% endhint %}

## 5. 수신 객체 지정 람다: with와 apply

수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 하는 것이 `수신 객체 지정 람다`.

### 5.1 with 함수

with 함수는 어떤 객체의 이름을 반복하지 않고도 그 객체에 대해 다양한 연산을 수행할 수 있다.

아래 두 코드는 일반으로 작성한 코드와 with 문으로 작성한 코드의 차이를 보여준다.

```kotlin
fun alphabet(): String {
    val result = StringBuilder()

    for(letter in 'A'..'Z'){
        result.append(letter)
    }

    return result.toString()
}
```

```kotlin
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {
        for(letter in 'A'..'Z'){
            this.append(letter)
        }
        this.toString()
    }
}
```

with의 첫번째 파라메터는 `StringBuilder`이고 두 번째 파라메터는 `람다`다.

람드를 괄호 밖으로 빼내는 관례를 사용함에 따라 전체 함수 호출이 언어가 제공하는 틀별 구문처럼 보인다.

with 함수는 첫번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 인자로 받은 람다 본문에서는 `this`를 사용하여 쓸 수 있다.

### 5.2 apply 함수

`apply`는 항상 저신에게 전달된 객체를 반환한다는 점뿐이다.

```kotlin
fun alphabet(): String = StringBuilder().apply {
    for(letter in 'A'..'Z'){
        this.append(letter)
    }
}.toString()
```

apply는 확장 함수로 되어있다. apply의 수신 객체가 전달받은 람다의 수신 객체가 된다.

apply 함수는 `객체의 인스턴스를 즉시 만들면서 득시 프로퍼티중 일부를 초기화해야 하는 경우 유용하다.`

`apply를 사용하면 어떤 객체라도 빌더 스타일의 API를 사용해 생성하고 초기화할 수 있다.`

## 요약

* 람다를 사용하면 코드 조각을 다른 함수에게 인자로 넘길 수 있다.
* 코틀린에서는 람다 함수 인자인 경우 괄호 밖으로 람다를 뺴낼 수 있고, 람다의 인자가 단 하나뿐인 경우 인자 이름을 짖어하지 않고 it 이라는 디폴트 이름으로 부를 수 있다.
* 람다 안에 있는 코드는 그 람다가 들어있는 바깥 함수의 변수를 읽거나 쓸 수있다,
* 메소드, 생성자, 프로퍼티의 이름 앞에 ::을 붙이면 각각에 대한 참조를 만들 수 있다. 그럼 참조르 ㄹ란다 대신 다른 함수에게 넘길 수 있다.
* filter, map, all, any 등의 함수를 활용하면 컬렉션에 대한 대부분의 연산을 직접 원소를 이터레이터하지 않고 조합할 수 있다.
* 시퀀스를 사용하면 중간 결과를 담는 컬렉션을 생성하지 않고도 컬렉션에 대한 여러 연선을 조합할 수 있다.
* 함수형 인터페이스를 인자로 받는 자바 함수를 호출할 경우 람다를 함수형 인터페이스 인자 대신 넘길 수 있다.
* 수신 객체 지정 람다를 사용하면 람다 안에서 미리 정해둔 수신 객체의 메소드를 직접 호출할 수 있다.
* 표준 라이브러리의 with 함수를 사용하면 어떤 객체에 대한 참조를 반복해서 언급하지 않으면서 그 객체의 메소드를 호출할 수 있다. apply를 사용하면 어떤 객체라도 빌더 스타일의 API를 사용해 생성하고 초기화할 수 있다.
