# ch7. 연산자 오버로딩과 기타 관계

## 1.산술 연산자 오버로딩

자바 에서는 원시 타입과 String 만 `+` 연산자 가 가능하지만 코틀린에서는 미리 정해진 이름의 함수를 연결해주는 기법을 `관례` 라고 하며 이를 이요하여 객체에 연산자 사용이 가능하다.

이번장에는 화면에 점을 표현하는 point 클래스를 예로든다.

### 1.1 이상 산술 연산 오버로딩

아래와 같이 Point 클래스에 연산자 오버로딩하는 함수 앞에 `operator` 키워드를 붙여 어떤 함수가 관계를 따르는 함수임을 명확히 할 수 있다.

```kotlin
data class Point(val x: Int, val y: Int) {
    operator fun Point.plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}
```

그럼 이제 아래와 같이 `+` 를 사요하면 위의 관례에 적용된 `plus`함수가 호출된다.

```kotlin
val point1 = Point(1, 1)
val point2 = Point(1, 2 )

val point3 = point1 + point2

print(point3)

>>Point(x=2, y=3)
```

{% hint style="info" %}
+ 연산자는 plus 함수 호출로 컴파일 된다.

a + b ---&gt; a.plus\(b\)
{% endhint %}

다른 함수도 똑같다. 회부 함수의 클래스에 대한 연산자를 정의할때는 관례를 따르는 이름의 확장 함수로 구현한다. 코틀린에서 정의할 수 있는 연산자 오버로딩의 할 수 있는 목록은 다음과 같다.

|  | 함수 이 |
| :--- | :--- |
| a \* b | times |
| a / b | div |
| a % b | mod \(1.1 부터 rem\) |
| a + b | plus |
| a - b | minus |

{% hint style="info" %}
연산자 우선순위는 언제나 표준 숫자 타입에 대한 연산자 우선순위와 같다.

### a + b \* c 의 식에서 \* 이 먼저 수행된다.
{% endhint %}

### 1.2 복합 대입 연산자 오버로딩

Plus 와 같은 연산자를 오버로딩하면 코틀린은 + 연산자뿐 아니라 그와 관련있는 연자인 `+=`도 자동으로 함께 지원한다. 이를 `복합 대입 연산자`라 부른다.

`Point += Point(3, 4)`는 Point = Point + Point\(3, 4\) 라고 쓴것과 같다. 물론 변수가 변경가능할 때만 그렇다.

`+=` 연산이 객체에 대한 참조를 다름 참조로 바꾸기보다 원래 객체의 내부 상태를 변경하게 만들고 싶을 떄가 있다. `변경 가능한` 컬렉션 원소를 추가하는 경우다.

```kotlin
val numbers = ArrayList<Int>()
numbers += 12
println(numbers[0])

>> 12
```

반환 타입이 Unit 인 plusAssign 함수를 정의하면 코틀린은 += 연산자에 그 함수를 사용한다.

비숫하게 `minusAssign`, `timesAssign` 등의 이름을 사용한다.

```kotlin
@kotlin.internal.InlineOnly
public inline operator fun <T> MutableCollection<in T>.plusAssign(element: T) {
    this.add(element)
}
```

클래스를 설계할 때 `plus` 와 `plusAssign` 을 동시에 추가하지 말라, 혼란을 야기할 수 있다. 변경 가능한 클래스를 설계하려면 `pliusAssign`을 변경 불가한 클래스는 `plus` 를 추가하여 새로운 값을 반환하는 연산만을 추가하는 것이 좋다.

{% hint style="info" %}
### +, - 는 항상 새로운 컬렉션을 반환하며

### +=, -= 연산자는 항상 변경 가능한 컬렉션에 작용해 메모리에 있는 객체 상태를 변화한다.
{% endhint %}

```kotlin
val list = ArratList<Int>()

list += 3 // list 를 변경

val newList = list + listOf(1, 2 ,3) // 새로운 객체 반환
```

### 1.3 단항 연산자 오버로딩

`-a` 와 같은 단항 연산자도 제공한다. 이것 또한 `operator`키워드를 이용해서 사용한다.

아래와 같이 연산자를 로버로딩 하여 단항 연산자를 사요할 수 있다.

```kotlin
operator fun Point.unaryMinus(): Point {
    return Point(-x, -y)
}
```

아래는 단항 연산자를 오버로딩 할 수 있는 목록이다.

| 식 | 함수 이 |
| :--- | :--- |
| +a | unaryPlus |
| -a | unaryMinus |
| !a | not |
| ++a, a++ | inc |
| --a, a-- | dec |

