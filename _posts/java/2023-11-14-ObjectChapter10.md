---
title:  "오브젝트 - 상속과 코드 재사용"
categories:
  - Java
---

우아한 테크코스 4주차 프리코스 미션인 크리스마스 문제를 풀고 리팩토링을 하기 위해서 'Chapter 10 - 상속과 코드 재사용'과 조영호 님의 '애플리케이션 아키텍처와 객체지향' 세미나를 참고했습니다. 처음 미션을 진행했을 때 중복되는 코드가 굉장히 많았는데 어떻게 중복되는 코드를 줄이고 추상화에 의존하는 방법에 대해서 알아보도록 하겠습니다.

## 상속과 중복 코드
중복 코드를 제거해야 하는 가장 큰 이유는 변경을 방해하기 때문입니다. 비즈니스 요구 사항은 매번 바뀌고 100% 작성한 모든 코드는 수정되기 마련입니다. 이번 프리코스 미션에서 작성한 코드를 예시로 중복 코드가 일으키는 문제점에 대해서 설명하겠습니다.

~~~java
// 특별 할인 정책
public class SpecialDiscountEvent {
    List<Integer> days = List.of(3, 10, 17, 24, 25, 31);
    private static final int BASE_DISCOUNT_AMOUNT = 1000;

    public int discountAmount(int dateOfVisit) {
        if (days.contains(dateOfVisit)) {
            return BASE_DISCOUNT_AMOUNT;
        }
        return 0;
    }
}
~~~
~~~java
// 크리스마스 할인 정책
public class ChristmasDiscountEvent {
    private static final Integer BASE_DISCOUNT_AMOUNT = 1000;

    public static int discountAmount(int dateOfVisit) {
        return BASE_DISCOUNT_AMOUNT + ((dateOfVisit-1) * 100);
    }
}
~~~
~~~java
// 평일 할인 정책
public class WeekdayDiscountEvent {
    List<Integer> days = List.of(3, 4, 5, 6, 7, 10, 11, 12, 13, 14, 17, 18, 19, 20, 21, 24, 25, 26, 27, 28, 31);

    private static final Integer BASE_DISCOUNT_AMOUNT = 2023;

    public int discountAmount(int dateOfVisit, int dessertCount) {
        if (days.contains(dateOfVisit)) {
            return BASE_DISCOUNT_AMOUNT * dessertCount;
        }
        return 0;
    }
}
~~~
~~~java
// STEP 7 : 혜택 내역 출력
int christmasDiscountAmount = getChristmasDiscountAmount();
int weekdayDiscountAmount = getWeekdayDiscountAmount();
int weekendDiscountAmount = getWeekendDiscountAmount();
int specialDiscountAmount = getSpecialDiscountAmount();
int giveDiscountAmount = getGiveDiscountAmount(giveMenu);
int totalDiscountAmount = christmasDiscountAmount + weekdayDiscountAmount + weekendDiscountAmount + specialDiscountAmount + giveDiscountAmount;

printBenefitDetails(totalDiscountAmount, weekdayDiscountAmount, christmasDiscountAmount, weekendDiscountAmount, specialDiscountAmount, giveDiscountAmount);
~~~

추가적으로 주말 할인 정책 등이 있습니다. 위 3개의 클래스 모두 discountAmount라는 중복 코드가 있습니다. 만약 세금을 부과한다는 요구사항이 추가된다면 5개의 클래스(+ 주말 할인, 증정 할인 클래스) 모두 수정해야 합니다. 그리고 추가 할인 정책이 들어온다면 전체 할인 금액을 출력하기 위해서 파라미터의 개수 또한 증가하게 됩니다.

이를 해결하기 위해 상속을 이용할 수 있지만 이번주 프리코스 미션에서는 상속을 사용할 경우 생기는 커플링 문제점을 보여드릴 수 없기 때문에 과감히 넘어가도록 하겠습니다. 자세한 예시는 오브젝트 책 p.318쪽을 참고하시면 됩니다.


## 추상화에 의존하고 차이를 메서드로 추출하라
커플링 문제를 없애기 위해서는 추상화를 이용하면 됩니다. 자식 클래스가 부모 클래스의 구현에 의존하도록 만드는 것이 아니라 부모 클래스와 자식 클래스 모두 추상화에 의존하도록 설계하면 됩니다.

~~~java
// 추상 클래스
public abstract class Discount {
    protected List<DiscountPolicy> policies;

    public Discount(DiscountPolicy ... policies) {
        this.policies = Arrays.asList(policies);
    }

    public abstract void calculateDiscountAndSaveDetail(BenefitDetail benefitDetail, OrderSheet orderSheet);
}
~~~
~~~java
// Discount 추상 클래스에 의존하는 크리스마스 할인 정책 클래스
public class ChristmasDiscount extends Discount{
    private static final Integer BASE_DISCOUNT_AMOUNT = 1000;
    private static final String DISCOUNT_NAME = "크리스마스 디데이 할인: -";

    public ChristmasDiscount(DiscountPolicy... policies) {
        super(policies);
    }

    @Override
    public void calculateDiscountAndSaveDetail(BenefitDetail benefitDetail, OrderSheet orderSheet) {
        for (DiscountPolicy policy : policies) {
            if (policy.isSatisfiedBy(orderSheet)) {
                benefitDetail.saveEvent(DISCOUNT_NAME, discountPrice(orderSheet.getDateOfVisit()));
            }
        }
    }

    private int discountPrice(int dateOfVisit) {
        return BASE_DISCOUNT_AMOUNT + ((dateOfVisit-1) * 100);
    }
}
~~~
~~~java
// Discount 추상 클래스에 의존하는 평일 할인 정책 클래스
public class WeekdayDiscount extends Discount {
    private static final String DISCOUNT_NAME = "평일 할인: -";
    private static final Integer BASE_DISCOUNT_AMOUNT = 2023;

    public WeekdayDiscount(DiscountPolicy... policies) {
        super(policies);
    }

    @Override
    public void calculateDiscountAndSaveDetail(BenefitDetail benefitDetail, OrderSheet orderSheet) {
        for (DiscountPolicy policy : policies) {
            if (policy.isSatisfiedBy(orderSheet)) {
                benefitDetail.saveEvent(DISCOUNT_NAME, discountPrice(orderSheet));
            }
        }
    }

    private int discountPrice(OrderSheet orderSheet) {
        return BASE_DISCOUNT_AMOUNT * orderSheet.getOrders()
                .entrySet()
                .stream()
                .filter(o -> o.getKey().getMenuType() == MenuType.DESSERT)
                .mapToInt(Map.Entry::getValue)
                .sum();
    }
}
~~~

Discount 추상화 클래스에 의존하게 만들어서 커플링 문제를 해결했지만 아직 calculateDiscountAndSaveDetail() 메서드가 모든 클래스에서 중복됩니다.

## 중복 코드를 부모 클래스로 올려라
위에서 calculateDiscountAndSaveDetail() 메서드에서 for문을 돌면서 할인 정책에 해당하는지 확인하는 과정은 변하지 않는 부분이고 세부 할인 금액을 계산하는 과정은 각 클래스마다 변하게 됩니다. 완전히 동일한 코드인 calculateDiscountAndSaveDetail() 메서드는 부모 클래스로 올리고 할인 금액을 구하는 세부 로직은 자식 클래스에서 오버라이딩할 수 있도록 protected로 선언하면 '추상화에 의존하고 중복된 코드를 제거'하는 최종 목표에 도달할 수 있습니다.

완성된 코드는 아래와 같습니다.

~~~java
public abstract class Discount {
    protected List<DiscountPolicy> policies;

    public Discount(DiscountPolicy ... policies) {
        this.policies = Arrays.asList(policies);
    }

    public void calculateDiscountAndSaveDetail(BenefitDetail benefitDetail, OrderSheet orderSheet) {
        policies.stream()
                .filter(discountPolicy -> discountPolicy.isSatisfiedBy(orderSheet))
                .forEach(discountPolicy -> calculateAndSave(benefitDetail, orderSheet));
    }

    abstract protected void calculateAndSave(BenefitDetail benefitDetail, OrderSheet orderSheet);
}
~~~
~~~java
public class ChristmasDiscount extends Discount{
    private static final Integer BASE_DISCOUNT_AMOUNT = 1000;
    private static final String DISCOUNT_NAME = "크리스마스 디데이 할인: -";

    public ChristmasDiscount(DiscountPolicy... policies) {
        super(policies);
    }

    @Override
    protected void calculateAndSave(BenefitDetail benefitDetail, OrderSheet orderSheet) {
        benefitDetail.saveEvent(DISCOUNT_NAME, discountPrice(orderSheet.getDateOfVisit()));
    }

    private int discountPrice(int dateOfVisit) {
        return BASE_DISCOUNT_AMOUNT + ((dateOfVisit-1) * 100);
    }
}
~~~
~~~java
public class WeekdayDiscount extends Discount {
    private static final String DISCOUNT_NAME = "평일 할인: -";
    private static final Integer BASE_DISCOUNT_AMOUNT = 2023;

    public WeekdayDiscount(DiscountPolicy... policies) {
        super(policies);
    }

    @Override
    protected void calculateAndSave(BenefitDetail benefitDetail, OrderSheet orderSheet) {
        benefitDetail.saveEvent(DISCOUNT_NAME, discountPrice(orderSheet));
    }

    private int discountPrice(OrderSheet orderSheet) {
        return BASE_DISCOUNT_AMOUNT * orderSheet.getOrders()
                .entrySet()
                .stream()
                .filter(o -> o.getKey().getMenuType() == MenuType.DESSERT)
                .mapToInt(Map.Entry::getValue)
                .sum();
    }
}
~~~

## 마치며
이번 글에서는 오브젝트 책에서 나온 내용을 토대로 4주차 프리코스 미션인 산타 게임을 풀어봤습니다. 상속만 사용한다면 코드를 재사용하고 있다는 느낌은 들지만 사실 중복 코드 제거를 위해 새로운 중복 코드를 만들어야 합니다. 또, 부모 클래스 코드를 재사용하기 위해 개발자가 세운 가정을 이해해야 하는 번거로운 과정을 거쳐야 합니다. 따라서 추상 클래스를 상속해야 하며, 그렇게 했을 경우에 생기는 이점에 대해서도 알아봤습니다.

마지막으로 이번 미션에서의 디펜던시를 그려보면서 마무리 하도록 하겠습니다.

![디펜던시](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/java/object/10장디펜던시.jpeg?raw=true)



