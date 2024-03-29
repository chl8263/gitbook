# OOP 의 7대 개념

## OOP 의 4가지 특성

* ### 캡슐화 (encapsulation)
  * 데이터와 그 데이터에 작용하는 메서드를 하나로 묶음
  * 정보 숨기기 : 개체 안에 있는 데이터를 외부로보터 보호
    * 전부 혹은 일부
    * 외부: 다른 클래스에 속한 개체들
* ### 상속 (inhheritance)
  * 이미 존재하는 개체를 기반으로 확장된 개체를 만드는 방법
  * 확장된 개체
    * 기존의 개체에 속한 데이터와 동작을 모두 물려 받음
    * 여기에 다른 데이터나 동작을 추가할 수 있음
  * 실용적인 용도: 코드 중복을 막음
    * 여러 개체에 공통되는 데이터와 동작을 부모 개체로 만듦
    * 여러 개체는 각각 그 부모 개체를 상속 받음
    * 그 후 자기에만 필요한 데이터나동작을 추가
  * 사람에게는 점진적 학습이 가장 효율적
* ### 다형성 (polymorphism)
  * 같은 지시를 내렸는데 다른 종류의 개체가 동작을 달리하는 것
  * 어떤 함수 구현이 실행될지는 실행 중에 결정됨
    * 이를 늦은 바인딩 이라고 한다.
  * 일반적인 함수 호출은 이른 바인딩
    * 컴파일 중에 결정됨
  * 다형성의 혜택을 받으려면 상속 관계가 필요
    * 부모 개체에서 함수 시그내쳐를 선언
    * 자식 개체에서 그 함수를 다르게 구현(overriding)
  * 보통 다형성이라하면 서브타입(subtype) 다형성 이라고 한다.
* ### 데이터 추상화 (abstraction)
  * 추상화
    * 수학: 일반화란 의미 (반의어: 구체화)
    * OOP: 개체 속에 있는 실제 데이터느 함수 구현 방법에 종속되지 않겠단 뜻
    * 다형성을 통한 추상화
    * 추상 클래스나 인터페이스를 사용하는 추상화
  * 데이터 추상화
    * 개체 사용 시 그 안에 정확이 어떤 데이터가 있는지 알 필요 없음
    * 개체 안에 있는 데이터에 직접 접근 불가
      * 그 대신 개체의 함수를 통해 접근
    * 즉, 캡슐화는 데이터 수상화는 이루는 방법 중 하나

## + OOP 의 7대 개념

* ### 연관  (association)
  * 어떤 개체가 제공하는 기능을 다른 개체가 이용하는 관계
  * 종종 상속과 비교해서 설명
    * 상속은 자식 개체가 부모 개체의 모든 것을 내포하는 관계
    * 연관은 한 개체가 다른 개체를 참조하는 관계
    * 실세계에서 개체들이 상호작용하는 모습은 보통 연관과 비슷
  * 세부적으로 다시 집합과 컴포지션으로 나누기도 함
    * 특히 UML 이란 다이어그램을 그릴 때
    * 이 셋을 구분하지 않고 다 합쳐서 컴포지션이라 하기도 함
* 컴포지션 (composition)
  * 합성, 조합, 조립, 구성 등 다양한 번역어가 존재
  * 여러 개의 부품(그 자체로 개체)을 조립해서 새 개체를 만드는 방법
  * 집합과의 차이
    * 부품 그 자체로는 존재 의의가 없음
    * 조립품이 소멸할 때 부품도 같이 소멸(즉, 부품은 조립품의 수명을 따름)
* 집합 (aggregation)
  * 여러 개체를 모아 다른 개체를 만들지만 별도로 존재 가능
  * 컴포지션과의 차이
    * 각 개체들이 따로따로 살아남을 수 있음
