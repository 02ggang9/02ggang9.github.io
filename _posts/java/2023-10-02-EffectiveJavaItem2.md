생성자에 넘겨주는 파라미터도 많고 여러 필드 중에 필수인 필드, 선택적인 필드가 있을텐데 반드시 필수인 애들"만"있으면 생성자로 넘겨주도록 "강제"할 수 있다.

생성자 체이닝, 점층적 생성자 패턴

점층적 생성자 패턴은 아래와 같다.
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

위의 코드처럼 체이닝을 걸 수 있다.

혹시 이렇게 체이닝을 하지 않았다면 점층적 생성자 패턴(체이닝)을 해보는 것도 고려해보자.

근데 체이닝의 문제점은 new NutritionFats() 이렇게 할 때 매개변수로 뭘 줘야할지 모름. 이떄 쓰는 단축키가 커맨드+p임. 이러면 각각의 생성자가 받는 타입을 알 수 있다. 근데 이거는 인텔리제이를 쓸 때 얘기임. 즉 파라미터가 많아지면 어디에 들어가는지도 모르고 어떤 값을 주는지도 헷갈린다!

---
두번째는 자바빈즈 패턴
세터를 사용하는 거임. 기본 생성자를 제공하고.

장점 : 객체 생성이 간단해진다! 기본생성자를 사용하기 때문에 넘겨줄 파라미터 값도 없고..
근데 필수적인 필드 값들이 셋팅이 안하고 넘길 수 있음. 불안정한 산태. 이걸 해결하려면 주석으로 문서화? 하는 방법밖에 없음.

그래서 필수 필드 값들은 생성자를 사용해서 "강제화"한다. 그리고 선택적인 필드 값들은 세터를 사용해서 넣는 두 방법을 혼용하는 방법을 사용하는 것도 괜찮다!

단점 : 불변 객체로 만들 수 없다! -> 프리징을 사용해서 완화할 수 있지만 잘 안쓰고 자바에서는 직접 구현해야함. 복잡도 up

---

빌더

자바빈즈의 표준스펙에 세터는 리턴 타입이 void, 하지만 빌더들은 빌더 그 자체를 리턴한다.
필수적인 필드 값은 생성자로 받고 선택적인 필드 값은 리턴 값이 Builder인 세터 비슷한 메서드를 사용한다.

플루언트 API == 메서드 체이닝 -> ... 찍으면서 이어갈 수 있는 것.

필수적인 필드 값들은 생성자로 강제화 하기 때문에 자바빈즈보다 안전한다. (consistency)하다!

빌더는 오히려 코드를 이해하기 어렵게 됨. 만들때 필드들이 중복이됨. 그래서
모든 경우에 빌더를 사용하면 안됨. 필수적인 필드가 있고 옵션적인 필드가 있는데 너무 많아서 생성자의 매개변수가 너무 늘어난다. 그리고 이뮤터블하게 만들고 싶다 -> 빌더를 쓰자!

롬복에 빌더를 쓰면 알아서 빌더를 만들어줌. 너무 간단함. 애노테이션 하나로 코딩을 해야할 게 없청 줄어듬.
단점은 모든 매개변수를 받는 생성자가 생김. -> 극복가능 이런 일이 발생하는 일이 싫으면 @Allargs~~ 어노테이션의 access level을 private로 해서 기본 생성자를 막을 수 있다! 그래서 밖에서는 무조건 빌더를 통해서 개발을 해야 한다.

또 다른 단점은 필수적으로 받는 값을 생성자로 못 받음.. 굉장히 어려움. 롬복 빌더에는 이런 기능이 없음.

또 다른 단점은 기본적인 빌더를 사용하다가 롬복의 빌더로 갈아끼우면 코드가 깨진다. 그런데 이것은 builderClassName으로 해결할 수 있다!
