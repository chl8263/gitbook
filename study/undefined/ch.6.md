# ch.6 메시지와 인터페이스

객체 설계의 핵심은 객체들이 주고받는 메시지이고 이를통해 클래스가 정의된다. 객체가 수신하는 메시지들이 객체의 퍼블릭 인터페이스를 구성한다. 휼륭한 퍼블릭 인터페이스를 얻기 위해서는 책임 주도 설계 방법을 따르는 것만으로는 부족하다. 유연하고 재사용 가능한 파블릭 인터페이스를 만드는 데 도움이 되는 설계 원칙과 기법을 적용하는것이 이번장의 핵심이다.

### 1.협력과 메시지

#### 클라이언트-서버 모델

협력은 객체간에 메시지를 이용하여 요청할 때 만들어 지는 것이다. 두 객체 사이의 협력 관계를 설명하기 위해 사용되는 전총적인 메타포는 **클라이언트-서버 모델** 이다. 협력 안에서 메시지를 전송하는 객체를 클라이언트, 메시지를 수신하는 객체를 서버라고 부른다. 협력은 클라이언트가 서버의 서비스를 요청하는 **단방향 상호작용** 이다.

#### 메시지와 메시지 전송

* 메시지\(message\) : 객체들이 협력하기 위해 사용할 수 있는 유일한 의사소통 수단
* 메시지 전송\(message sending\), 메시지 패싱\(message passing\) : 한 객체가 다른 객체에게 도움을 요청하는 것
* 메시지 전송자\(message sender\) : 메시지를 전송하는 객체
* 메시지 수신자\(message receiver\) : 메시지를 수신하는 객체

> 서버-클라이언트 모델에서는 메시지 전송자를 클라이언트, 수신자를 서버 라고 한다.

메시지는 오퍼레이션면\(operation name\) 과 인자\(argument\)로 구성되며 메시지 전송은 여기에 메시지 수신자를 추가한 것이다.

```java
--수신자--  --오퍼레이션명--  ---인자--- 
condition.isStisfiedBy(screening);
```

#### 메시지와 메서드

위의 코드처럼 수신자에 해당하는 오퍼레이션이 메시지이다. 메시지를 실제로 수행하는 함수 또는 프로시저를 **메서드** 라고 한다. 즉, 동일한 메시지 일지라도 객체의 타입에 따라 실행되는 메서드는 다를 수 있다.\(다형성\)

메시지와 메서드의 구분은 메시지 전송자와 메시지 수신자가 느슨하게 결합될 수 있게 한다. 메시지 전송자는 어떤 메시지만을 전송해야 하는지만 알면 되고 메시지 수신자 역시 누가 보냈는지 알 필요가 없다. 메시지 수신자는 메시지가 도착했다는 사실만 알면 된다. 메시지 수신자는 메시지를 처리하기 위해 필요한 메서드를 결정할 수 있는 자율권을 누린다.

#### 퍼플릭 인터페이스와 오퍼레이션

* 퍼플릭 인터페이스 : 객체가 의사소통을 위해 외부에 공개하는 메시지의 집합
* 오퍼레이션 : 퍼블릭 인터페이스에 포함된 메시지

> 오퍼레이션은 구현이 아닌 추상화다. 반면 UML에서의 메서드는 오퍼레이션을 구현한 것이다. 즉, 메서드는 오퍼레이션에 대한 구현이고 퍼플릭 인터페이스와 오퍼레이션의 관점에서 볼 때 '메서드 호출' 보다 '오퍼레이션 호출' 이 더 적합하다.

#### 시그니처

* 시그니처 : 오퍼레이션이나 메서드의 명세를 나타낸 것으로, 이름과 인자의 목록을 포함한다. 대부분의 언어는 시그니처의 일부로 반환 타입을 포함하지 않지만 반환타입을 시그니처의 일부로 포함하는 언어도 존재함.

### 2.인터페이스와 설계품질

좋은 인터페이스란 최소한의 인터페이스와 추상적인 인터페이스 라는 조건을 만족해야한다. 이를 만족하기 위해서는 **책임 주도 설계 방법** 을 따르는것이다.

퍼블릭 인터페이스의 품질에 영향을 미치는 원칙과 기법

#### 1.디미터 법칙

내부 구조에 강하게 결합되지 않도록 협력 경로를 제한하라는 것. "낮선 자에게 말하지 말라" 또는 "오직 인접한 이웃하고만 말하라" 로 요약할 수 있다.

예를들어 Java, C\# 같이 \(.\) 도트를 이용해 메시지를 전송하는 언어에서는 **"오직 하나의 도트만을 사용하라"** 라는 말로 요약된다.

디미터의 법칙을 따르기 위해 클래스가 특정 조건을 만족하는 대상에게만 메시지를 전송하도록 프로그램이 해야한다.

* this 객체
* 메서드의 매개변수
* this의 속성
* this의 속성인 컬렉션의 요소
* 메서드 내에서 생성된 지역 객체

```java
public class ReservationAgency{
    public Reservation reserve(Screening screening, Customer customer, int audienceCount) {
        Money fee = screening.calculate(audienceCount);
        return new Reservation(customer, screening, fee, audienceCount);
    }
}
```

위의 코드는 ReservationAgency 가 Screening의 어떠한 내부 정보를 알지 못하고 즉, 내부 구조에 결합돼 있지 않기 때문에 Screening의 내부 구현을 변경할 때 ReservationAgency를 함께 변경할 필요가 없다.

이를 **부끄럼타는 코드\(shy code\)**라 말하고, 불필요한 어떤 코드도 외부에 제공하지 않고 대른 객체의 구현에 의존하지 않는 코드를 말한다.

디미터의 법칙을 위반하는 코드의 전형적인 예 - **기차충돌**

```java
screening.getMovie().getrDiscountCondition();
```

#### 2.묻지 말고 시켜라

디미터 법칙은 휼륭한 메시지는 객테의 상태에 관해 묻지 말고 원하는 것을 시켜야 한다는 사실을 강조함.

메시지를 수신하는 객체의 상태를 송신자가 절대 제어해서는 안되고 수신자가 직접 제어하게끔 시켜는 것이 바람직하다.

#### 3. 의도를 드러내는 인터페이스

메서드를 명명하는 두 가지 방법이 있다.

* 어떻게 수행하는지를 중점으로 작명한 메서드

```java
public class PeriodCondition {
	public boolean isStisfiedByPeriod(){
		...
	}
}

public class SequenceCondition {
	public boolean isStisfiedBySequence(){
		...
	}
}
```

위의 두 class 안의 메서드는 **모두 할인 조건을 판단하는데 동일한 작업을 수행 하지만 메서드의 이름이 다르기 때문에 두 메서드의 내부 구현을 정확히 이해하고 있지 않으면 같은 작업을 한다고 생각하기가 어렵다** 또한 이 메서드들은 **클라이언트들로 하여금 객체의 종류를 꼭 알아야 하게끔 만든다.**

위처럼 어떻게 수행하는지를 드러내는 이름은 메서드의 내부 구현을 먼저 고려 하고, 설계 시작부터 클래스의 내부 구현에 관해 고민할 수 밖에 없다.

* 무엇을 수행하는지를 중점으로 작명한 메서드

위의 코드를 클라이언트 관점에서 두 메서드는 할인 여부를 판단하기 위해 사용되므로 둘다 **isStisfiedBy** 로 바꿔주는 것이 바람직하다.

```java
public class PeriodCondition {
    public boolean isStisfiedBy(){
        ...
    }
}

public class SequenceCondition {
    public boolean isStisfiedBy(){
        ...
    }
}
```

이제 두 메서드는 동일한 목적을 가진 메서드 라는것을 알 수 있지만, 이름이 같다고 해서 동일한 메시지를 처리할 수 있는것은 아니므로 해당 메서드의 클래스를 동일한 계층타입으로 묶는것이 필요하다.

```java
public interface DiscountCondition {
    public boolean isStisfiedBy();
}
```

```java
public class PeriodCondition implement DiscountCondition {
    public boolean isStisfiedBy(){
        ...
    }
}

public class SequenceCondition implement DiscountCondition {
    public boolean isStisfiedBy(){
        ...
    }
}
```

무엇을 수행하는지를 중점으로 작명한 메서드는 목적을 먼저 생각하도록 만들며, 결과적으로 협력하는 클라이언트의 의도에 부합하도록 메서드의 이름을 짓게 된다. 이를 **의도를 드러내는 선택자** 라고한다.

#### 인터페이스에 의도를 드러내자

메서드 이름은 클라이언트가 원하는 방향을 정확하게 표현해야한다.

```java
public class TicketSeller {
    public void sellTo(Audience audience) {....} // 클라이언트는 TicketSeller이고 TicketSeller는 Audience에게 어떤것을 판다. 클라이언트 관점에서 명명
}

public class Audience {
    public void buy(Ticket ticket) {....} // Audience는 Ticket을 산다.
}

public class Bag {
    public void hold(Ticket ticket) {....} // Bag은 Ticket을 가지고 있어야한다.
}
```

이 방법은 **묻지말고 시켜라** 스타일을 지키게끔 한다.

### 3.원칙의 함정

설계는 트레이드오프의 산물이다.

초보자들은 원칙을 맹목적으로 추종한다. 심지어 초보자를 구분하는 가장 중요한 기준이라고 할 수 있다. 초보자는 원칙을 맹목적으로 추종한다. 원칙이 현재 상황과 부적합하다고 판단된다면 과감하게 원칙을 무시하라. 원칙을 아는것 보다 더 중요한 것은 언제 원칙이 유용하고 언제 유용하지 않은지를 판단할 수 있는 능력을 기르는 것이다.

디미터 법칙은 도트를 하나만 쓰게 강제하는 것은 아니다.

```java
Intstream.of(1, 15, 20, 3, 9).filter(x -> x > 10).distinct().count();
```

위의 코드는 디미터의 법칙을 위배하지 않는다. 디미터 법칙은 결합도와 관련된 것이며, 이 결합도가 문제가 되는 것은 객체의 내부 구조가 외부로 노출되는 경우로 한정된다. 기차충돌처럼 보이지만 객체의 내부 구현에 대해 어떤 정보도 노출하지 않으면 그것은 디미터 법칙을 준수한 것이다.

#### 결합도와 응집도의 충돌

영화 예매 시스템의 PeriodCondition 클래스에서 isStisfiedBy 메서드는 screening에게 질의한 상영 시작 시간을 이용해 할인 여부를 결정한다. 이코드는 Screening의 내부 상태를 가져와서 사용하기 때문에 캡슐화를 위반한 것으로 보인다.

```java
public class PeriodCondition implement DiscountCondition {
    public boolean isStisfiedBy(){
        return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
            ....
    }
}
```

따라서 screening 에게 책임을 옮겨서 캡슐화를 하는것이 더 좋아보인다.

```java
public class Screening {

    ...

    public boolean isDiscountable(...){
        return startTime.dayOfWeek.equals(dayOfWeek) &&
            ....
    }
}

public class PeriodCondition implement DiscountCondition {
    public boolean isStisfiedBy(){
        return screening.isDiscountable(...);
    }
}
```

위의 코드처럼 하면 Screening이 할인조건 계산을 떠안게 된다. 하지만 **Screening의 본질적인 책임은 할인조건 계산이 아니라 영화를 예매하는것이다.**

이런경우 이전의 코드를 이용하여 캡슐화를 향상시키는것 보다 Screening의 응집도를 높이고 \(Screening의 본질이 아닌 할인조건 계산 메서드가 들어오면 응집도가 낮아짐\) Screening과 PeriodCondition의 결합도를 낮추는 것이 전체적으로 더 좋은 방법이다.

> 소프트웨어의 설계에 법칙이란 존재하지 않는다. 원칙을 맹신하지 마라. 설계는 트레이트오프의 산물이다. 소프트웨어 설계에 존재하는 몇 안되는 법칙 중 하나는 **"경우에 따라 다르다"** 이다.

### 4.명령-쿼리 분리 원칙

명령-쿼리 분리 원칙은 퍼블릭 인터페이스에 오퍼레이션을 정의할 때 참고할 수 있는 지침을 제공한다.

* 루틴 : 어떤 절차를 묶어 호출 가능하도록 이름을 부여한 기능 모듈
* 프로시저 : 부수효과를 발생시킬 수 있지만 반환값은 없음
* 함수 : 부수효과를 발생시킬 수 없고 반환할 수 있음

**명령**과 **쿼리**는 객체의 인터페이스 측면에서 프로시저와 함수를 부르는 또 다른 이름이다.

* 명령 : 객체의 상태를 수정하는 오퍼레이션
* 쿼리 : 객체와 관련된 정보를 반환하는 오퍼레이션

명령과 쿼리 둘중에 하나의 역할을 하는 오퍼레이션을 만드는 것이 중요하다. 두개를 합쳐버리면 부수효과가 일어난다.

