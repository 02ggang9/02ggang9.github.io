---
published: true
title: "Effective Java Item 2 - 생성자에 매개변수가 많다면 빌더를 고려하라"
categories:
  - Java
---
생성자에 매개변수가 많다면 생기는 문제점과 여러 해결 방안에 대해서 알아보도록 하겠습니다.

## 생성자 문제점
생성자에 매개변수가 많으면 인스턴스를 생성할 때 어디에 어떤 값을 넣어야 하는지 혼란이 옵니다. 물론 IntelliJ의 단축키(Command + P)를 사용해 타입과 파라미터의 이름을 알 수 있지만 다른 IDE를 사용한다면 피할 수 없는 문제입니다.

## 해결 방안 1 - 자바빈즈 패턴
자바빈즈 패턴에 setter 메서드를 사용해서 생성자 문제점을 해결할 수 있습니다. 기본 생성자로 인스턴스를 생성하고 setter 메서드로 필드 값들을 채우는 방식입니다. 장점은 기본 생성자로 쉽게 인스턴스를 생성할 수 있고 코드의 가독성이 올라갑니다. 하지만 치명적인 단점으로는 일관성(consistency)을 보장할 수 없고 불변성 문제가 있습니다. 아래의 코드는 실수로 필수 값인 servingSize를 빼먹어 일관성을 깬 예시입니다.

~~~java
public class NutritionFacts {
    private int servingSize = -1; // 필수
    private int servings = -1; // 필수
    private int calories = 0; // 선택
    private int fat = 0; // 선택
    private int sodium = 0; // 선택

    public NutritionFacts() {}

    // 자바 빈즈 규약에 맞는 setter 메서드들..
    public void setServingSize(int val) {
        this.servingSize = val;
    }
}
~~~

~~~java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
...
~~~

위의 문제를 해결하기 위해서 아래처럼 생성자를 만들어 필수 값들을 넣도록 "강제" 할 수 있습니다.

~~~java
public class NutritionFacts {
    private int servingSize = -1; // 필수
    private int servings = -1; // 필수
    private int calories = 0; // 선택
    private int fat = 0; // 선택
    private int sodium = 0; // 선택

    public NutritionFacts(int servingSize, int servings) {
        this.servingSize = servingSize;
        this.servings = servings;
    }

    // 자바 빈즈 규약에 맞는 setter 메서드들..
    public void setServingSize(int val) {
        this.servingSize = val;
    }
}
~~~

~~~java
NutritionFacts cocaCola = new NutritionFacts(240, 8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
...
~~~

위 코드처럼 생성자를 통해서 일관성 문제를 일부 해결할 수 있지만 불변성 문제는 해결할 수 없습니다. 이를 해결하기 위해서 빌더 패턴이 나왔습니다.

## 해결 방안 2 - 빌더 패턴
필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻고 선택 매개변수들은 일종의 setter 메서드를 이용해 값을 설정할 수 있습니다. 이렇게 함으로써 일관성 문제와 불변성 문제를 해결하고 메서드 체이닝으로 가독성 또한 확보할 수 있습니다. 아래는 빌더 패턴 코드입니다.

~~~java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;
        
        // 선택 매개변수
        private final int calories = 0;
        private final int fat = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            this.calories = val;
            return this;
        } 

        public Builder fat(int val) {
            this.fat = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.calories = builder.calories;
        this.fat = builder.fat;
    }
}
~~~

위의 코드를 보면 꽤나 복잡하고 필드들이 중복되지만 롬복을 사용하면 코드를 훨씬 간결하게 사용할 수 있습니다.

~~~java
@Builder
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
}
~~~

롬복의 단점으로 모든 매개변수를 받는 생성자가 생기는 문제가 있는데 생성자가 생기는 것이 싫다면 @AllArgsConstrutor(access = AccessLevel.PRIVATE)로 설정해 외부에서 생성자 사용을 막을 수 있습니다.

## 마치며
> API는 시간이 지날수록 매개변수가 많이지는 경향이 있음을 명심하자. 애초에 빌더로 시작하는 편이 나을 때가 많다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다. (Effective Java p.22)


생성자의 파라미터 개수가 많으면 생기는 문제점과 해결 방안으로 자바빈즈 패턴, 빌더 패턴에 대해서 알아봤습니다. 중간 자바 빈즈 규약에서 기본 생성자를 사용하지 않는 코드를 작성했는데 자바빈즈 규약에서는 기본 생성자를 권장하고 있습니다. 이처럼 부가적인 내용은 다음 포스팅에서 알아보도록 하겠습니다.

