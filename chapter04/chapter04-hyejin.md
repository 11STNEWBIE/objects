## 4. 설계 품질과 트레이드오프  

앞선 챕터에서 구현했던 영화 예매 시스템을 책임 중심 설계가 아닌 데이터 중심 설계를 통해 구현한 코드를 살펴보기

그리고 둘의 차이점을 통해 책임 할당 원칙에 대해 더 자세히 이해하기

### 01 데이터 중심의 영화 예매 시스템

- 영화 객체에 저장될 데이터들은 무엇이 있을까?
```java

    public class Movie {
        private String title;
        private Duration runningTime;
        private Money fee;
        private List<DiscountCondition> discountConditions;
    
        private MovieType movieType;
        private Money discountAmount;
        private double discountPercent;
    
    ...
```

객체의 상태 또한 구현에 속한다는 점 → 구현은 불안정하기 때문에 변하기 쉽다

또한 구현에 관한 세부사항이 객체의 인터페이스에 스며들게 되어 캡슐화의 원칙이 무너진다!

- 최종적으로 예매하기 클래스는... (일부만)
```java
    for(DiscountCondition condition : movie.getDiscountConditions()) {
                if (condition.getType() == DiscountConditionType.PERIOD) {
                    discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
                            condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                            condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
```
😨

### 02 설계 트레이드오프

- 캡슐화
    - 구현과 인터페이스
        - 구현 : 변경될 가능성이 높은 부분
        - 인터페이스 : 상대적으로 안정적인 부분
    - 객체지향에서 제일 중요한 원리이다!
    - 캡슐화는 변경 가능성이 높은 부분을 객체 내부로 숨기는 추상화 기법
        - 변경될 수 있는 *어떤 것* 이라도 추상화해야 한다.
- 응집도와 결합도
    - 응집도
        - 모듈에 포함된 내부 요소들이 연관되어 잇는 정도
        - 하나의 목적을 위해 긴밀히 협력한다면 높은 응집도
        - 변경의 관점에서는, 변경이 발생할 때 모듈 내부에서 발생하는 변경의 정도
    - 결합도
        - 어떤 모듈이 다른 모듈에 대해 꼭 필요한 지식만 알고 있다면 두 모듈은 낮은 결합도
        - 변경의 관점에서는, 한 모듈이 변경되기 위해 다른 모듈의 변경을 요구하는 정도
    - 높은 응집도, 낮은 결합도 → 좋은 설계
    - 즉, 응집도와 결합도는 변경과 관련된 것

### 03 데이터 중심 영화 예매 시스템의 문제점

- 캡슐화 위반
    - 다시, 변경될 수 있는 *어떤 것* 이라도 추상화해야 한다.
    - setFee나 getFee를 통해 fee에 접근한다면 캡슐화 위반이 아닌가? 그렇지 않다
    - fee라는 이름의 인스턴스 변수가 있다는 사실이 노출된다.
    - 협력에 대해 고민하지 않고 무수한 getter와 setter를 만들면 캡슐화를 위반하게 될 수밖에 없다
- 높은 결합도

    case PERCENT_DISCOUNT:
                        discountAmount = movie.getFee().times(movie.getDiscountPercent());

fee의 타입이 변경된다면 → getFee 메서드의 반환 타입 수정 → 관련된 모든 구현 수정...

- 낮은 응집도
    - 어떤 코드를 수정한 후에 아무 상관도 없던 코드에 문제가 발생하는 것은 응집도가 낮을 때 발생하는 대표적인 증상!
    - 단일 책임 원칙(SRP)

### 04 자율적인 객체를 향해

** 속성의 가시성을 private으로 설정했다고 해도 접근자와 수정자를 통해 속성이 외부로 제공된다면 캡슐화 위반이다!

- 객체를 설계할 때는

    1) 이 객체가 어떤 데이터를 포함해야 하고

    2) 이 객체가 데이터에 대해 수행해야 하는 오퍼레이션은 무엇인지

       둘 다 고민해야 한다.

### 05 하지만 여전히 부족하다

- 캡슐화의 진정한 의미
    - 캡슐화가 단순히 객체 내부의 데이터를 외부로부터 감추는 것이 아니라
    - 변경될 수 있는 어떤 것 (영화 예제에서는 할인 정책의 종류) 이라도 감추는 것

### 06 데이터 중심 설계의 문제점

- 본질적으로 너무 이른 시기에 데이터에 대해 결정하도록 강요한다
    - 데이터 또한 구현의 일부임
- 협력이라는 문맥을 고려하지 않고 객체가 고립된 상태에서 오퍼레이션을 결정함
