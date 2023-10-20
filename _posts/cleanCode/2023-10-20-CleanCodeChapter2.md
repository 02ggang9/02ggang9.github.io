---
published: false
title:  "CleanCode - 의미 있는 이름"
categories:
  - cleanCode
---

## 의도를 분명히 밝혀라

의도를 분명히 밝히기 위해서는 좋은 '이름'을 작성하면 됩니다. 분명한 의도는 코드 리뷰를 할 때 큰 힘을 발휘합니다. 메서드가 어떤 동작을 하는지, 변수가 뜻하는 의미는 무엇인지를 알 수 있다면 코드 이해가 쉬워지고 시간을 절약할 수 있습니다.

CleanCode 책에서는 이름을 지을 때 몇 가지 고려사항을 제시합니다. 

1. 변수의 존재 이유는 무엇인가?
2. 함수의 수행 기능은 무엇인가?
3. 함수의 사용 방법은 무엇인가?

위의 세 가지 질문을 답변하지 못하고 주석으로 대체한다면 의도를 분명히 드러내지 못 한 것입니다.

~~~java
// Before
int d; // 경과 시간(단위: 날짜)

// After
int elapsedTimeInDays;
int daysSinceCreation;
~~~

위의 예시에서 변수 d는 아무런 의미도 드러나지 않습니다. 따라서 측정하려는 값(time)과 단위를 표현하도록 변경한다면 분명한 의도를 밝힐 수 있습니다.

~~~java
// Before -> 함수의 수행 기능을 명확히 알 수 없음
public List<int[]> getThem() {
    List<int[]> list1 = new ArrayList<int[]>();
    for (int [] x : theList) {
        if(x[0] == 4) {
            list1.add(x)
        }
    }
    return list1;
}

// After(Version1) -> 함수가 수행하는 기능을 명확히 밝힘
public List<int[]> getFlaggedCells() {
    ...
}

// After(Version2) -> 인덱스 0과 숫자 4가 의미하는 바를 상수 값을 이용해 명확히 밝힘
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for (int[] cell : gameBoard) {
        if(cell[STATUS_VALUE] == FLAGGED) {
            ...
        }
    }
}


// After(Version3) -> isFlagged라는 명시적인 함수를 사용해 FLAGGED라는 상수를 감출 수 있음
public List<int[]> getFlaggedCells() {
    List<int[]> flaggedCells = new ArrayList<int[]>();
    for (int[] cell : gameBoard) {
        if(isFlagged()) {
            ...
        }
    }
}
~~~

위의 예시처럼 메서드의 이름과 매직 넘버를 상수 값으로 바꿔 프로그래머의 의도를 정확히 파악할 수 있게 되었습니다. 

<br>

## 상수 값과 Enum 사용
위의 예시에서 봤듯이 매직 넘버를 상수 값으로 바꿔 프로그래머의 의도를 정확히 파악할 수 있었습니다. 그래서 우아한테크코스 1주차 프리코스를 진행하면서 모든 문자열과 숫자를 상수 값으로 바꾸려는 강박이 생겼습니다.

### 상수 값 사용

~~~java
// Before
private void checkRestartNumberValidation(String restartNumber) {
    if (restartNumber.equals("1") || restartNumber.equals("2")) {
        return;
    }
    throw new IllegalArgumentException();
}

// After -> 상수 값 사용으로 의도를 명확히 밝힘
public static final String RESTART_NUMBER = "1";
public static final String END_NUMBER = "2";

private static void isRestartNumberValid(String inputNumber) {
    if (inputNumber.equals(RESTART_NUMBER) || inputNumber.equals(END_NUMBER)) {
        return;
    }

    throw new IllegalArgumentException();
}
~~~

한판의 게임이 끝나고 클라이언트가 1을 입력하면 게임 재시작, 2를 입력하면 게임 완전 종료가 될 수 있도록 요구사항을 제시해 주셨습니다. 이 코드를 작성한 프로그래머는 요구사항을 알고 있기 때문에 의도를 파악할 수 있지만 다른 프로그래머가 본다면 매직넘버가 의미하는 바를 알 수 없습니다.

### Enum 사용

~~~java
public enum Message {

    START_GAME_MESSAGE("숫자 야구 게임을 시작합니다"),
    GUESS_NUMBER_MESSAGE("숫자를 입력해주세요 : "),
    END_GAME_MESSAGE("3개의 숫자를 모두 맞히셨습니다! 게임 종료"),
    RESTART_GAME_MESSAGE("게임을 새로 시작하려면 1, 종료하려면 2를 입력하세요."),
    ;


    private final String message;

    Message(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}

public class OutputView {
    public static void printEndGame() {
        System.out.println(END_GAME_MESSAGE.getMessage());
    }

    public static void printRestartGame() {
        System.out.println(RESTART_GAME_MESSAGE.getMessage());
    }
}
~~~

위처럼 Enum을 사용해 OutputView에서 사용할 메시지를 모아둔다면 상수 값 이름으로 출력할 메시지의 의도를 알 수 있고, 메시지 수정 요구가 들어와도 손쉽게 해결할 수 있습니다. 

### 모든 매직 넘버를 상수 값으로?

모든 문자열과 숫자를 상수 값으로 변경하지 않아도 됩니다. 프리코스를 진행하면서 모두 상수 값으로 변경하려고 했지만 G25: 매직 숫자는 명명된 상수로 교체하라(p.386)에 글을 읽으면서 생각을 바꿨습니다. 코드 자체가 명확하고 이해하기 쉽다면 상수를 뒤로 숨길 필요가 없습니다.

~~~java
// WORK_HOURS_PER_DAY
int dailypay = hourlyRate * 8;

// SECONDS_PER_DAY
86,400

// LINES_PER_PAGE
55
~~~

책에서는 위의 예시로 설명합니다. 첫번째 에시는 WORK_HOURS_PER_DAY라는 상수를 사용해도 괜찮겠지만 숫자 8이 들어간 공식은 너무 깔끔하기 때문에 상수를 추가하기 꺼려진다고 합니다. 반면에 86,400이나 55는 의미하는 바가 모호하기 때문에 상수 뒤로 숨기는 것이 코드를 이해하는데 도움을 줄 수 있다고 합니다.

## 마치며
변수 이름, 메서드 이름, 클래스 이름 등을 잘 지음으로써 얻는 이득이 많으므로 항상 좋은 이름을 짓도록 노력합시다. 이름을 지을 때 책에서 제시한 가이드라인을 잘 지키고 있는지 생각하고 문자열과 숫자가 나온다면 항상 상수 값으로 변경하지 말고 꼭 바꿔야 하는지 한번 쯤은 생각해보록 합시다.

