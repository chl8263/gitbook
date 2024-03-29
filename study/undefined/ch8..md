# ch8. 의존성 관리하기

### 1.의존성 이해하기

#### 변경과 의존성

어떤 객체가 협력하기 위해 다른 객체를 필요로 할 때 두 객체 사이에 의존성이 존재하게 된다. 의존성은 싱행 시점과 구현 시점에 서로 다른 의미를 가진다.

* 실행 시점 : 의존하는 객체가 정상적으로 동작하기 위해서는 실행 시에 의존 대상 객체가 반드시 존재해야 한다.
* 구현 시점 : 의존 대상 객체가 변경될 경우 의존하는 객체도 함께 변경된다.

어떤 객체가 예정된 작업을 정상적으로 수행하기 위해 다른 객체르 필요로 하는 경우 두 객체 사이에 의존성이 존재한다고 말한다.

의존성은 항상 단반향이며 역은 성립하지 않음.

#### 의존성 전이

* 의존성 전이 : 객체가 의존하는 대상에 대해서도 자동적으로 의존하게 된다는것. 의존성이 실제로 전이될지 여부는 변경의 방향과 캡슐화의 정도에 따라 달라짐. 의존성 전이는 변경에 의해 널리 전파될 수도 있다는 경고일 뿐이다.
* 직접 의존성 : 한 요소가 다른 요소에 직접 의존하는 경우
* 간접 의존성 : 직접적인 관계는 존재하지 않지만 의존성 전이에 의해 영향이 전파되는 경우를 가리킨다.

#### 런타임 의존성과 컴파일타임 의존성

* 런타임 의존성 : 어플리케이션이 실행되는 시점에 대한 의존성, 다루는 주체는 객체 (객체의 최종 인스턴스를 다루어야 한다)
* 컴파일 타임 의존성 : 코드를 컴파일 하는 시점의 의존성, 다루는 주체는 클래스 (객체의 인스턴스 및 추상 클래스를 다룬다)

유연하고 재사용 가능한 코드를 설계하기 위해서는 두 종류의 의존성을 서로 다르게 만들어야 한다.

> 어떤 클래스의 인스턴스가 다양한 클래스의 인스턴스와 협력하기 위해서는 협력할 인스턴스의 구체적인 클래스를 알아서는 안된다. 실제로 협력할 객체가 어떤 것인지는 런타임에 해결해야 한다. 따라서 컴파일타임 구조와 런타임 구조 사이의 거리가 멀면 멀수록 설계가 유연해지고 재사용 가능해진다.

#### 컨텍스트 독립성

어떤 클래스를 사용할때 구체적인 클래스를 명시하지 않고 상위 클래스로 명시하여 컴파일 타임 의존성을 어떤 런타임 의존성으로 얼마나 대체하냐에 따라 최소한의 가정만으로 이뤄져 있다면 다른 문맥안에서 재사용하기가 더 수월해진다. 이를 **컨텍스트 독립성**이라고 한다.

```java
public AA (PercentDiscountPolicy policy) {
    ...
}

// 위 코드처럼 특정 문맥에 강하게 결합하지 않고 아래처럼 상위클래스를 파라메터로 받아 재사용이 용이하게 만드는것.

public AA (Policy policy){
    ...
}
```

#### 의존성 해결하기

위의 예제 코드처럼 컴파일 타임 의존성을 적절한 런타임 의존성으로 교체하는 것을 **의존성 해결**이라고 부른다.

의존성 해결을 위한 3가지 방법

* 객체를 생성하는 시점에 생성자를 통해 의존성 해결
* 객체 생성후 setter 메서드를 통해 의존성 해결 // 객체를 생성하고 변경을 열어놓고 싶을떄 사용. 단점: 불안정함 // 생성자와 setter를 둘 다 열어놓으면 안정적이고 유연하다.
* 메서드 실핼 시 인자를 이용해 의존성 해결    // 의존 대상이 매번 달라져야 할 경우

### 유연한 설계

#### 의존성과 결합도

객체지향 패러다임의 근간의 협력이다. 객체들이 협력하기 위해서는 서로의 존재와 수행 가능한 책임을 알아야 한다. 이런 지식들이 객체 사이의 의존성을 낳는다.

의존성은 객체들의 협력을 가능하게 만드는 매개체라는 관점에서는 바라직한 것이다. 하지만 과하면 문제가됨.

의존성의 정도에 따라 문제를 나누는 것인데, 바람직한 의존성은 재사용성과 관련이 있다. 의존성을 상위 클래스로 두어 재사용성을 높이는 것이 바람직한 설계임.

다른 환경에서 재사용하기 위해 내부 구현을 변경하게 만드는 모든 의존성은 바람직하지 않은 의존성이다. 바람직한 의존성이란 컨텍스트에 독립적인 의존성을 의미하며 다양한 환경에서 재사용될 수 있는 가능성을 열어놓는 의존성을 의미한다.

> 어떤 두 요소 사이에 존재하는 의존성이 바람직할 때 두 요소가 느슨한 결합도 또는 약한 결합도를 가진다고 말한다. 반면 두 사이에 의존성이 바람직하지 못할 때 단단한 결합도 또는 강한 결합도를 가진다고 말한다.

#### 지식이 결합을 낳는다.

결합도의 정도는 한 요소가 자신이 의존하고 있는 다름 요소에 대해 할고 있는 정보의 양으로 결정된다. 한 요소가 다른 요소에 대해 더 많은 정보를 알고 있을수록 두 요소는 강하게 결합된다.

더 많이 알수록더 많이 결합된다. 더 많이 알고 있다는 것은 더 적은 컨텍스트에서 재사용 사능하다는 것을 의미한다.

결합도를 느슨하게 만들기 위해서는 협력하는 대상에 대해 필요한 정보 외에는 최대한 가추는 것이 중요하고 우리는 이것을 **추상화**로 목표를 달성한다.

#### 추상화에 의존하라

추상화 : 어떤 양상, 세부사항, 구조를 좀 더 명확하게 이해하기 위해 특정 절차나 물체를 의도적으로 생략하거나 감춤으로써 복잡도를 극복하는 방법.

```
결합도가 느슨해짐 
    |          - 구체 클래스 의존성
    |          - 추상 클래스 의존성
    |          - 인터페이스 의존성
   \ /
    V
```

**의존하는 대상이 더 추상적일수록 결합도는 더 낮아진다**

#### new는 해롭다

'new' 키워드를 이용해 인스턴스를 생성하면 결합도가 극단적으로 높아지는 2가지 이유.

* new 연산자를 사용하기 위해서는 구체 클래스의 이름을 직접 기술해야 한다. 따라서 new 를 사용하는 클라이언트 추상화가 아닌 구체 클래스에 의존할 수밖에 없기 때문에 결합도가 높아진다.
* new 연산자를 생성하려는 구체 클래스뿐만 아니라 어떤 인자를 이용해 클래스의 생성자를 호출해야 하는지도 알아야 한다. 따라서 new를 사용하면 클라이언트가 알아야 하는 지식의 양이 늘어나기 때문에 결합도가 높아진다.

#### Java 표준 클래스는 생성해도 무방하다.
