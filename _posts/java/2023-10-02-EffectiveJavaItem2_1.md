---
published: false
title: "Effective Java Item 2 - 생성자에 매개변수가 많다면 빌더를 고려하라"
categories:
  - Java
---

## 해결 방안 1 - 점층적 생성자 패턴
점층적 생성자 패턴은 코드로 보면 한번에 이해할 수 있습니다.

~~~java
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;

    public NutritionFacts(int servingSize, int servings) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = 0;
        this.fat = 0;
        this.sodium = 0;
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = 0;
        this.sodium = 0;
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = 0;
    }
~~~

~~~java
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings,0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0)
    }
~~~

자바 빈 스팩은 옛날에 만들어진 스팩임.

게터와 세터 이름의 규약을 정의를 해놨음. 인텔리제이에서도 게터와 세터를 자바빈 스펙에 맞게 만들어준다.

boolean은 어떻게 만들어질까?
~~~java
private boolean healthy;

public boolean isHealthy() {
    return healythy;
}
~~~

이렇게 규약이 정해져야지 GUI 툴들도 규약에 따라서 값을 가져올 수 있다.

그리고 기본 생성자를 만드는 것을 자바 빈즈 규약에서 권장한다. 이래야 편함.

Serializable 인터페이스?
객체를 직렬화하고 역직렬화 해줌. 그리고 저장하고 읽고 쓸 수 있게 해주겠다! 객체를 저장 가능한 정보로 만들어야 함. 그 상태 그대로 저장했다가 그대로 복원을 하게 해주는 기능이다.

게터와 세터의 이름은 자바 빈 규약에 맞는 이름으로 꼭 만들어야 한다.

자바의 레코드?