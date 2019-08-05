- 구현을 할때, 클래스가 아닌 객체에 초점을 맞출 때에만 객체지향 패러다임으로의 전환을 할 수 있음

  1. 어떤 클래스가 필요한지 고민하기 전에 어떤 객체들이 필요한지 고민하라.

     ```html
     - 객체와 클래스의 차이
     	1. Class
         객체를 만들어 내기 위한 설계도
     		연관되어 있는 변수와 메소드의 집합
     	2. Obejct
     		구현할 대상
     		클래스에 선언된 모양 그대로 생성된 실체
     ```

  2. 객체를 독립적인 존재가 아니라 기능을 구현하기 위해 협력하는 공동체의 일원으로 본다.(객체는 다른 객체에게 도움을 주거나 의존하면서 살아가는 협력적인 존재.)

- 도메인(domain)

  - 문제를 해결하기 위해 사용자가 프로그램을 사용하는 분야를 domain이라고 함

- 클래스를 구현할 때 주목할 점은 인스턴스 변수의 가시성은 private이고 메소드의 가시성은 public이다. 가장 중요한 점은 어떤 것을 외부에 공개하고 어떤 것을 감출지 결정하는 것이다.
  - 대부분 속성에 접근을 막고, public 메소드를 통해서만 객체 내부의 상태를 변경하게 해야됨.
- 객체가 상태와 행동을 함께 가진다는 복합적인 존재.
- 객체가 스스로 판단하고 행동하는 자율적인 존재.
  - 데이터와 기능을 한 덩어리로 묶음(캡슐화)로 문제를 해결하게 함.
    - public Interface : 외부에서 접근 가능한 부분
    - implementation : 외부에서 접근 불가하고 내부에서만 접근 가능한 부분
  - ISP : 인터페이스와 구현의 분리 원칙은 객체지향 프로그램을 만들기 위한 핵심
- 객체의 상태는 숨기고 행동만 외부에 공개하는 것이 핵심





- 객체는 다른 객체의 인터페이스에 공개된 행동을 수행하도록 request하고, request받은 객체는 자율적으로 처리하여 response한다.

  - 상호 작용을 할 방법은 send a message하는 것뿐. receive한 메세지를 처리하기 위한 자신만의 방법을 method라고 한다.

  ==> message, method의 구분에서부터 polymorphism의 개념이 출발한다.



- 부모 클래스에 기본적인 흐름을 구현하고 중간에 필요한 처리를 자식 클래스에게 위임하는 디자인 패턴을 TEMPLATE METHOD패턴이라고 한다.



- 컴파일 타임 의존, 런타임 의존

  - 코드의 의존성과 실행 시점의 의존성이 다르면 다를수록 코드 이해하기 어려워짐
  - 코드의 의존성과 실행시점의 의존성이 다르면 코드는 더 유연해지고 확장 가능해짐

- 상속은 기존 클래스를 기반으로 새로운 클래스를 쉽고 빠르게 추가할 수 있는 간편한 방법 제공.

  - 부모 클래스와 다른 부분만을 추가해서 새로운 클래스를 쉽고 빠르게 만드는 방법을 차이에 의한 프로그래밍이라고 한다.

- upcasting : 자식클래스가 부모클래스를 대신하는 것.

- 다형성 : 실제로 어떤 메소드가 실행될 것인지는 메시지를 수신하는 객체의 클래스가 무엇이냐에 따라 달라진다.

  - 메시지와 메소드를 실행시점에 바인딩한다 --> lazy binding(dynamic binding)

- 추상화 장점

  1. 상위 정책을 쉽고 간단하기 표현하여 기본적인 어플리케이션의 협력 흐름을 기술.
  2. 기존 구조를 수정하지 않고도 새로운 기능을 쉽게 추가하고 확장 가능.

- 코드 재사용

  - 상속은 코드를 재사용하기 위해 널리 사용되는 방법
  - 합성이 더 좋은 방법이다라는 설이 많음. 합성은 다른 객체의 인스턴스를 자신의 인스턴스 변수로 포함해서 재사용하는 방법.
  - 상속은 캡슐화 위반, 유연하지 못한 설계를 초래한다는 것이 단점.(컴파일 심점에 관계를 결정) --> 실행 시점에 객체 종류 변경이 불가능

- 인터페이스에 정의된 메시지를 통해서만 코드를 재사용하는 방법을 합성이라고 함.

  - 구현을 효과적으로 캡슐화 가능(인터페이스에 정의된 메시지를 통해서만 재사용이 가능하기 때문)
  - 상속은 클래스를 통해 강하게 결합되는 데 비해 합성은 메시지를 통해 느슨하게 결합 --> 따라서, 상속보다 합성을 선호함.
  - 코드를 재사용하는 경우에는 상속보다 합성을 선호하는 것이 옳지만, 다형성을 위해 인터페이스를 재사용하는 경우 상속과 합성을 함께 조합해서 사용해야됨(트레이드 off 필요)

  ```text
  Dependency Injection – is a composition of structural design patterns, in which for each function of the application there is one, a conditionally independent object (service) that can have the need to use other objects (dependencies) known to it by interfaces. Dependencies are transferred (implemented) to the service at the time of its creation. This is a situation where we introduce an element of one class into another. In practice, DI is implemented by passing parameters to the constructor or using setters. Libraries that implement this approach are also called IoC containers.
  ```

  

