## 05. 책임 할당하기  

- 그래서 어떻게 하라고?
    - 어떤 객체에 어떤 책임을 할당하지 알기 쉽지 않다
    - GRASP 패턴 (General Responsibility Assignment Software Pattern) by Craig Larman

### 01 책임 주도 설계를 향해

- 데이터보다 행동을 먼저 결정하라
    - 이 객체가 수행해야 하는 책임은 무엇인가?
    - 이 책임을 수행하는 데 필요한 데이터는 무엇인가?
- 협력이라는 문맥 안에서 책임을 결정하라
    - 책임은 객체의 입장이 아니라 *객체가 참여하는 협력* 에 적합해야 한다
    - 메시지 전송자에게 적합한 책임을 할당해야 한다 (수신자가 아니다)
    - 메시지를 결정한 후에 객체를 선택해야 한다
        - 임의의 객체가 메시지를 수신할 것이라는 사실을 믿고, 자신의 의도를 표현한 메시지를 전송
        - 메시지 수신자에 대한 어떠한 가정도 할 수 없다 → 메시지 수신자 캡슐화

### 02 책임 할당을 위한 GRASP 패턴

- 도메인 개념에서 출발하기
    - 개념들의 의미와 관계가 정확하거나 완벽할 필요는 없다
    - 출발점이 필요할 뿐
    - 필요한 것은 도메인을 그대로 투영한 모델 (X) 구현에 도움이 되는 모델 (O)
- 정보 전문가에게 책임을 할당하라
    - INFORMATION EXPERT (정보 전문가) 패턴
        - 책임을 수행할 정보를 알고 있는 객체에게 책임 할당하기
        - 주의할 점 : 정보 전문가가 데이터를 반드시 *저장* 하고 있을 필요는 없다.
    - 영화 예제
        - 예매하라 메시지는 Screening에게
        - 예매하려면 영화 가격을 계산해야 → 가장 많이 알고 있는 객체는 Movie
        - Movie에게 가격 계산하라는 메시지 전송
        - 할인 요금 계산해야 → 가장 많이 알고 있는 객체는 DiscountCondition → 할인 여부 판단하라는 메시지 전송
- 높은 응집도와 낮은 결합도
    - 할인 요금 계산 부분에서 Movie와 DiscountCondition 대신  Screening과 DiscountCondition이 협력하면 어떨까?
        - LOW COUPLING, HIGH COHESION 패턴 (책임과 협력의 품질 검토)
        - Movie와 DiscountCondition은 이미 결합되어 있으므로, 응집도와 결합도 관점에서 Movie가 협력하는 것이 더 좋다.
- 창조자에게 객체 생성 책임을 할당하라
    - 예매의 최종 결과물은 Reservation 인스턴스 생성
    - CREATOR 패턴 : 객체를 생성할 책임을 어떤 객체에게 할당할 것인가?
        - 객체 A의 생성 책임을 아래 조건을 최대한 많이 만족하는 객체 B에게 할당한다.
            - B가 A를 포함하거나 참조
            - B가 A를 기록
            - B가 A를 긴밀하게 사용
            - B가 A를 초기화하는 데 필요한 데이터를 가지고 있다

### 03 구현을 통한 검증

- DiscountCondition 개선하기 ⭐️
    - 일반적으로 설계를 개선하는 작업은 변경의 이유가 하나 이상인 클래스를 찾는 것으로부터 시작
    - 인스턴스 변수가 초기화되는 시점을 살펴보자
        - DiscountCondition의 경우
            - 순번 조건 표현시 : dayOfWeek, startTime, endTime은 초기화 X
            - 기간 조건 표현시 : sequence는 초기화 X
            - 함께 초기화되는 속성을 기준으로 코드를 분리해야 한다
        - 모든 메서드가 객체의 모든 속성을 사용한다면 클래스의 응집도는 높다
    - 클래스 응집도 판단
        - 하나 이상의 이유로 변경되어야 한다면 응집도가 낮은 것 / 변경의 이유를 기준으로 클래스 분리
        - 초기화하는 시점에 서로 다른 속성들을 초기화하고 있다면 응집도가 낮은 것 / 초기화되는 속성의 그룹을 기준으로 클래스를 분리
        - 메서드 그룹이 속성 그룹을 사용하는지 여부로 나뉜다면 응집도가 낮은 것 / 이들 그룹을 기준으로 클래스를 분리
    - 다형성을 통해 분리
        - DiscountCondition을 SequenceCondition과 PeriodCondition으로 분리
        - 다형성을 통해 DiscountCondition 인터페이스를 implements하도록
            - Movie는 구체적인 할인 조건에 대해서는 알지 못해도 된다.
                - 변경으로부터 보호 (PROTECTED VARIATIONS)
        - Movie 클래스도 동일하게, 단 구현을 공유해야 하므로 추상 클래스를 활용하도록 변경한다.

### 04 책임 주도 설계의 대안

- 일단 빠르게 목적한 기능을 수행한 코드를 작성한 후 리팩토링한다
    - 단, 코드를 수정한 후에 겉으로 드러나는 동작이 바뀌어서는 안 된다
    - 이것을 보장하기 위해 아마 Test의 작성이 중요할 것이다. (내 생각)

- 일단 작성한 후 응집도 높은 메소드들로 잘게 분해하기

```java
    public class ReservationAgency {
        public Reservation reserve(Screening screening, Customer customer,
                                   int audienceCount) {
            Movie movie = screening.getMovie();
    
            boolean discountable = false;
            for(DiscountCondition condition : movie.getDiscountConditions()) {
                if (condition.getType() == DiscountConditionType.PERIOD) {
                    discountable = screening.getWhenScreened().getDayOfWeek().equals(condition.getDayOfWeek()) &&
                            condition.getStartTime().compareTo(screening.getWhenScreened().toLocalTime()) <= 0 &&
                            condition.getEndTime().compareTo(screening.getWhenScreened().toLocalTime()) >= 0;
                } else {
                    discountable = condition.getSequence() == screening.getSequence();
                }
    
                if (discountable) {
                    break;
                }
            }
    
            Money fee;
            if (discountable) {
                Money discountAmount = Money.ZERO;
                switch(movie.getMovieType()) {
                    case AMOUNT_DISCOUNT:
                        discountAmount = movie.getDiscountAmount();
                        break;
                    case PERCENT_DISCOUNT:
                        discountAmount = movie.getFee().times(movie.getDiscountPercent());
                        break;
                    case NONE_DISCOUNT:
                        discountAmount = Money.ZERO;
                        break;
                }
    
                fee = movie.getFee().minus(discountAmount).times(audienceCount);
            } else {
                fee = movie.getFee();
            }
    
            return new Reservation(customer, screening, fee, audienceCount);
        }
    }
```

```java
    public class ReservationAgency {
        public Reservation reserve(Screening screening, Customer customer,
                                   int audienceCount) {
            boolean discountable = checkDiscountable(screening);
            Money fee = calculateFee(screening, discountable, audienceCount);
            return createReservation(screening, customer, audienceCount, fee);
        }
    
        private boolean checkDiscountable(Screening screening) {
            return screening.getMovie().getDiscountConditions().stream()
                    .anyMatch(condition -> condition.isDiscountable(screening));
        }
    
        private Money calculateFee(Screening screening, boolean discountable,
                                   int audienceCount) {
            if (discountable) {
                return screening.getMovie().getFee()
                        .minus(calculateDiscountedFee(screening.getMovie()))
                        .times(audienceCount);
            }
    
            return  screening.getMovie().getFee();
        }
    
        private Money calculateDiscountedFee(Movie movie) {
            switch(movie.getMovieType()) {
                case AMOUNT_DISCOUNT:
                    return calculateAmountDiscountedFee(movie);
                case PERCENT_DISCOUNT:
                    return calculatePercentDiscountedFee(movie);
                case NONE_DISCOUNT:
                    return calculateNoneDiscountedFee(movie);
            }
    
            throw new IllegalArgumentException();
        }
    
        private Money calculateAmountDiscountedFee(Movie movie) {
            return movie.getDiscountAmount();
        }
    
        private Money calculatePercentDiscountedFee(Movie movie) {
            return movie.getFee().times(movie.getDiscountPercent());
        }
    
        private Money calculateNoneDiscountedFee(Movie movie) {
            return movie.getFee();
        }
    
        private Reservation createReservation(Screening screening,
                                              Customer customer, int audienceCount, Money fee) {
            return new Reservation(customer, screening, fee, audienceCount);
        }
    }
```

- 그리고 메소드들을 적절한 클래스로 이동해 보자.
    - 객체를 자율적으로 만들기
