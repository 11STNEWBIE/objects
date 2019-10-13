# CHAPTER 09. 유연한 설계

8장에서 소개한 기법들을 원칙이라는 관점에서 정리해봅시다

## 01. 개방-폐쇄 원칙

> **개방-폐쇄 원칙(Open-Closed Principle, OCP)**: 
소프트웨어 개체(클래스, 모듈, 함수 등등)는 **확장**에 대해 열려 있어야 하고, **수정**에 대해서는 닫혀 있어야 한다.

- 확장에 열림: 요구사항 변경 시 새로운 '동작' 추가해 애플리케이션의 기능을 확장할 수 있음
- 수정에 닫힘: 기존 '코드' 수정 없이 애플리케이션에 동작을 추가/변경할 수 있음

어떻게 할까??

### 컴파일 의존성은 고정시키고 런타임 의존성을 변경하라

- 컴파일타임 의존성: 코드에서 드러나는 클래스 사이의 관계
- 런타임 의존성: 실행시에 협력에 참여하는 객체 사이의 관계

유연하고 재사용 가능한 설계에서 위 두 의존성은 서로 다른 구조를 가짐.

이 때 컴파일타임 의존성은 유지하면서 런타임 의존성의 가능성을 확장하고 수정할 수 있는 구조는 의존성의 관점에서 개방-폐쇄 원칭 원칙을 따른다고 볼 수 있음.

### 추상화

개방-폐쇄 원칙의 핵심은 **추상화**에 **의존**하는 것! 추상화를 하면 공통적인 부분은 문맥이 바뀌더라도 수정할 필요가 없어야 한다. 즉, 추상화 부분은 수정에 닫혀 있으며 추상화를 통해 생략된 부분은 확장의 여지를 남긴다.

이 때 추상화를 통해 수정에 폐쇄적으로 만들려면, 변하는 것과 변하지 않는 것이 무언지를 이해하고 이를 추상화의 목적으로 삼아야 함.

## 02. 생성 사용 분리

> **생성과 사용의 분리(separation use from creation)**: 
소프트웨어 시스템은 (응용 프로그램 객체를 제작하고 의존성을 서로 "연결"하는) 시작단계와 (시작 단계 이후에 이어지는) 실행 단계를 분리해야 한다.
``` java
public class Movie {
  ...
  private DiscountPolicy discountPolicy;

  private Movie(String title, Duration runningTime, Money fee) {
    ...
    this.discountPolicy = new AmountDiscountPolicy(...);
  }

  public Money calculateMovieFee(Screening screening) {
    return fee.minus(discountPolicy.calculateDiscountAmount(screeing);
  }
}
```

위 코드에서, `Movie` 의 할인 정책을 바꾸려면?  `AmountDiscountPolicy` 의 인스턴스를 생성하는 부분을 `PercentDiscountPolicy` 의 인스턴스를 생성하도록 직접 코드를 수정해야 함. 이는 개방-폐쇄 원칙을 위반.

하나의 클래스 안에서 객체의 생성과 사용이라는 두가지 이질적인 목적을 가진 코드가 공존하는 것이 문제. 유연하고 재사용 가능한 설계 원한다면 객체와 관련된 이 두 책임을 서로 다른 객체로 분리해야 한다.

어떻게?

### 클라이언트에서 객체를 생성

보편적인 방법은 객체 생성 책임을 클라이언트로 옮기는 것. 즉, `Movie` 의 클라이언트가 적절한 `DiscountPolicy` 인스턴스를 생성해 `Movie` 에게 전달하는 것(아래 코드 참고).
``` java
public class Client {
  public Money getAvatarFee() {
    Movie avatar = new Movie("아바타",
                              Duration.ofMinutes(120),
                              Money.wons(10000),
                              new AmountDiscountPolicy(...));
    return avatar.getFee();
  }
}
```

`AmountDiscountPolicy` 의 인스턴스 생성하는 책임을 클라이언트에게 맡겨 컨텍스트와 관련된 정보는 클라이언트로 옮기고 `Movie` 는 오직 `DiscountPolicy` 인스턴스를 사용하기만 함.

### 팩토리 추가하기

위에서 `Client` 역시 생성과 사용의 책임을 함께 지니고 있음. 그런데 `Movie` 를 사용하는 `Client` 도 특정한 컨텍스트에 묶이지 않기를 바란다면?

동일한 방법으로 문제를 해결할 수 있음. 즉 `Clinet`의 인스턴스를 사용할 문맥을 결정할 클라이언트로 `Movie` 생성 책임을 옮기는 것. 이 경우 객체 생성과 관련된 책임만 전담하는 별도의 객체를 추가하고 `Client` 는 이 객체를 사용하도록 만들 수 있다. 이처럼 생성과 사용을 분리하기 위해 객체 생성에 특화된 객체를 **FACTORY** 라고 부른다.
``` java
public class Factory {
  public Movie createAvatarMovie(){
    return new Movie("아바타",
                      Duration.ofMinutes(120),
                      Money.wons(10000),
                      new AmountDiscountPolicy(...));
  }
}

public class Client {
  private Factory factory;

  public Client(Factory factory) {
    this.factory = factory;
  }

  public Money getAvatarFee() {
    Movie avatar = factory.createMadMaxMovie();
    return avatar.getFee();
  }
}
```
**FACTORY** 를 사용하면 `Movie` 와 `AmountDiscountPolicy` 를 생성하는 책임 모두 **FACTORY** 로 이동할 수 있음. `Client` 는 사용과 관련된 책임만 남음. 즉, FACTORY를 통해 생성된 `Movie` 객체를 얻고, `Movie` 를 통해 가격을 계산하는 것.

### 순수한 가공물에게 책임 할당하기

> **순수한 가공물(PURE FABRICATION)**: 
책임을 할당하기 위해 창조되는, 도메인과 무관한 인공적인 객체
**FACTORY** 처럼 도메인 모델에 속하지 않으나 기술적인 결정으로서 만들어진 객체를 **PURE FABRICATION** 이라고 부른다.

이게 필요한 이유? 책임 할당의 기본 원칙은, 책임 수행에 필요한 정보 가장 많이 알고 있는 **INFORMATION EXPERT**(ex 도메인 모델)에게 책임을 할당하는 것. 그러나 도메인 개념을 표현하는 객체에게 책임을 할당하는 것만으로는 부족한 경우 생김(모든 책임을 도메인 객체에 할당하면 낮은 응집도, 높은 결합도, 재사용성 저하와 같은 문제 생길 수도).

따라서 이러한 측면에서 객체지향이 실세계의 모방이라는 말은 옳지 않음. 객체지향 애플리케이션은 도메인 개념뿐만 아니라 설계자들이 임의로 창조한 인공적인 추상화들이 포함되어 있음!

## 03. 의존성 주입

> **의존성 주입(Dependency Injection)**:
사용하는 객체가 아닌 외부의 독립적인 객체가 인스턴스를 생성한 후 이를 전달해서 인스턴스를 해결하는 방법

생성과 사용을 분리하면 `Movie` 는 인스턴스를 사용하는 책임만 남으며, 외부의 다른 객체로부터 인스턴스를 전달받아야 한다. 즉 의존성 주입이 필요.

의존성 주입은 의존성을 해결하기 위해 의존성을 객체의 퍼블릭 인터페이스에 명시적으로 드러내서 외부에서 필요한 런타임 의존성을 전달할 수 있도록 만드는 방법을 포괄하는 명칭으로, 다음 세 가지 방법을 가리키는 용어가 있음.

- **생성자 주입(constructor injection)**:  객체 생성 시 생성자를 통한 의존성 해결
- **setter 주입(setter injection)**: 객체 생성 후 세터를 통한 의존성 해결
- **메서드 주입(method injection)**: 메서드 실행 시 인자를 이용한 의존성 해결

각 방법의 장단점은 8장과 동일하니 참고ㅎ

### 숨겨진 의존성은 나쁘다

의존성 주입 외에도 의존성을 해결하는 방법 중 하나로 **SERVICE LOCATOR** 패턴이 있음. **SERVICE LOCATOR** 는 의존성을 해결할 객체들을 보관하는 일종의 저장소로, 객체가 직접 **SERVICE LOCATOR**에게 의존성 해결 요청.
``` java
public class Movie {
  ...
  private DiscountPolicy discountPolicy;

  private Movie(String title, Duration runningTime, Money fee) {
    ...
    this.discountPolicy = ServiceLocator.discountPolicy();
  }
}

public class ServiceLocator() {
  private static ServiceLocator soleInstance = new ServiceLocator();
  private DiscountPolicy discountPolicy;

  public static DiscountPolicy discountPolicy() {
    return soleInstance.discountPolicy;
  }

  public static void provide(DiscountPolicy discountPolicy) {
    soleInstance.discountPolicy = discountPolicy;
  }

  private ServiceLocator() {}
}
```

`Movie` 인스턴스가 `AmountDiscountPolicy` 의 인스턴스에 의존하기 바란다면, `ServiceLocator` 에 인스턴스를 등록한 후 `Movie` 를 생성하면 됨. 

``` java
ServiceLocator.provide(new AmountDiscountPolicy(...));
Movie avatar = new Movie("아바타",
                          Duration.ofMinutes(120),
                          Money.wons(10000));
```
그러나 **SERVICE LOCATOR** 패턴의 가장 큰 단점은! 의존성을 감춘다는 것. 바로 위 코드에서 `Movie` 인스턴스를 생성하는 부분의 코드만 마주한다고 했을 때, 의존성과 관련된 부분이 다 해결되었는지 런타임 가서야 발결될 수 있기 때문에(ex. NPE)..

또한 단위 테스트 작성도 어려움. `ServiceLocator` 는 내부적으로 정적 변수를 사용해 객체들을 관리하기 때문에 모든 단위 테스트 테스트에 걸쳐 `ServiceLocator`의 상태를 공유함 → 각 단위 테스트는 서로 고립되어야 한다는 원칙 위반.

**SERVICE LOCATOR** 패턴을 사용해야 하는 경우? 의존성 주입을 지원하는 프레임워크를 사용하지 못하는 경우, 혹은 깊은 호출 계층에 걸쳐 동일한 객체를 계속해서 전달해야하는 경우, 어쩔 수 없이..?

## 04. 의존성 역전 원칙

## 05. 유연성에 대한 조언
