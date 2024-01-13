---
published: true
title:  "Spring Batch - BDD 내부 메일링 서비스 개발 (1) - JavaMailSender를 통한 메일 발송과 마크다운 문법 지원, 메일 포맷팅"
categories:
  - spring
---

## 서론

제가 속해 있는 BDD(부산 개발자 소모임)는 명예회원이라는 제도가 있습니다. 명예회원은 실무에 종사하고 계시는 선배님들이고, 세미나 발표와 커피챗, 슬랙에서의 정보 공유 활동을 BDD와 같이 하시고 계십니다. 명예회원님들께 한달에 한 번씩 메일로 BDD 활동 내역과 발생했던 일(뉴스레터)을 보내드려 지속으로 BDD에 관심을 가지실 수 있게 메일링 시스템을 만들어 보라고 한상곤 교수님께서 말씀을 해 주셨습니다. 말씀을 듣자마자 Spring Batch를 공부하고 간단한 메일링 프로그램을 만들어 봤습니다. 

이번 포스트에서는 JavaMailSender를 통한 메일 발송과 간단한 마크다운 문법 지원, 포맷팅을 맞추기 위한 과정을 담았습니다. 다음 포스트에서는 성능 개선에 관한 내용으로 찾아뵙겠습니다. 아래는 완성된 메일링 서비스의 깃허브 주소입니다.

[BDD-CLUB/Bulk-mailing-service Github](https://github.com/BDD-CLUB/bulk-mailing-service)

## JavaMailSender를 사용하기 위한 환경 설정

Google 계정 관리 > 2단계 인증 설정 후 yml 파일 작성

[2단계 인증 설정 방법 최신](https://goodteacher.tistory.com/8)

[2단계 인증 설정 방법 최신 - stackOverflow](https://stackoverflow.com/questions/72930539/javax-mail-authenticationfailedexception-535-5-7-8-username-and-password-not-ac)

[yml 파일 작성 방법](https://github.com/BDD-CLUB/bulk-mailing-service#3-applicationyml-%ED%8C%8C%EC%9D%BC-%EC%B6%94%EA%B0%80)

첫 번째 링크와 두 번째 링크는 2단계 인증 설정을 위해서 제가 참고한 글입니다. 대부분 다른 글들은 과거 정책이 바뀌기 전이고 위의 두 글은 최신의 글입니다. 또, yml 파일 작성은 제가 만든 메일링 서비스의 Quick Start 코드입니다. fix라고 되어 있는 부분을 수정해서 사용하시면 됩니다. (password는 stack-overflow 글에서 보이는 무작위 16문자열을 입력해셔야 합니다.)

## JavaMailSender

JavaMailSender는 Spring Boot에서 제공하는 JavaMail을 확장한 인터페이스입니다. 기본적인 default 메서드로 핵심 기능을 제공하고 있어 간단히 텍스트, HTML, 이미지 등을 보낼 수 있습니다.

~~~java
@Service
@RequiredArgsConstructor
public class EmailService {

    private final JavaMailSender javaMailSender;

    public void sendEmail(String to, String subject, String text) {
        SimpleMailMessage emailMessage = new SimpleMailMessage();
        emailMessage.setTo(to);
        emailMessage.setSubject(subject);
        emailMessage.setText(text);
        javaMailSender.send(emailMessage);
    }

}
~~~

## 볼드체 지원 및 깔끔한 HTML 포맷팅 제공

위의 코드는 단순한 텍스트만 전송할 수 있는 문제점이 있습니다. 단순한 문자만으로는 기업에서 발송하는 전문적인 메일 느낌을 낼 수 없었습니다. 고민을 하던 중 우연히 메일함에서 "우아한테크 뉴스레터"를 보게 되었습니다. 우아한테크 뉴스레터는 아래와 같이 우아한형제들의 로고와 깔끔한 볼드체, 블로그 리다이렉트로 구성되었습니다.

![우아한테크 뉴스레터](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/batch/블로그글/우아한뉴스레터.png?raw=true)

우아한테크 뉴스레터의 HTML 포맷을 사용한다면 좀 더 전문적인 느낌을 낼 수 있다고 생각해서 개발자 도구를 열어 HTML 태그를 복사하고 보내고자 하는 텍스트를 파싱해서 전송하도록 설계했습니다. 또한, {} 중괄호로 감싸면 기본적인 볼드체로 나올 수 있도록 하는 문법도 제공했습니다. 작성한 코드는 아래의 Github 링크를 통해서 확인하실 수 있습니다.

[BDD 뉴스레터 Github Link](https://github.com/BDD-CLUB/bulk-mailing-service/blob/e1fa6e688f3462f7d000ad1394ac410383aa2a14/src/main/java/io/springbatch/springbatch/service/EmailService.java)

정규표현식을 사용해서 {} 중괄호 안에 있는 문자열을 "strong" 태그로 감싸도록 했습니다. 1차적으로 완성된 메일 서비스로 메일을 전송할 때 이미지는 아래와 같습니다.

![1차 뉴스레터](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/batch/블로그글/1차뉴스레터.png?raw=true)

## 마크다운 문법 제공

시간이 지나 BDD 임원진분들의 추가적인 요구사항이 생겼습니다. 메일로 보내려는 것이 글과 사진이기 때문에 추가적으로 사진을 보낼 수 있는 기능을 만들어야 했습니다. 이 추가 요구사항을 듣고 과거 한상곤 교수님의 공학작문및발표 수업에서 사내 문서 변환 도구(예를 들어, 마크다운에서 PDF로)를 만들어 사용하셨다는 말씀이 기억났습니다. 마크 다운 문법 중 제목3(볼드체) 문법과 이미지 첨부 문법을 제공한다면 간편하게 메일을 작성하실 수 있을거라 생각해 중괄호 문법을 삭제하고 마크 다운 문법으로 변경했습니다.

[BDD 뉴스레터 마크다운 지원 Github Link](https://github.com/BDD-CLUB/bulk-mailing-service/blob/f007ec659a4e149b3d9cfecb9b9eaeb3c449939d/src/main/java/io/springbatch/springbatch/bdd/email/entity/MdFormatConverter.java)

위의 코드에서도 정규 표현식을 통해서 "###"문법을 사용할 경우 볼드 처리를 했고, "\!\[\]\(\)" 문법을 사용할 경우 img 태그를 통해 이미지를 삽입하도록 변경했습니다. 아래의 예시처럼 md 형식으로 글을 작성하면 아래의 형식으로 이메일이 전송하게 됩니다.

~~~md
### BDD Mailing Service 1차 테스트

안녕하세요. 부산 개발 동아리 BDD입니다.

BDD 뉴스레터를 이용해주시는 선배님들께 진심으로 감사드립니다.

아래는 2024년 1월 BDD의 활동 내역들입니다.

### 메인페이지 시안 작성

![메인페이지 시안](https://file.notion.so/f/f/aaedcf79-6b31-4898-9a1f-5e2ad8ae925e/572fb2d5-3374-4e26-bd5f-65c17b11986f/%ED%94%84%EB%A6%AC%EC%A0%A0%ED%85%8C%EC%9D%B4%EC%85%9814.png?id=1f34b78c-5b64-4797-bb6c-6bd69eeeb0c1&table=block&spaceId=aaedcf79-6b31-4898-9a1f-5e2ad8ae925e&expirationTimestamp=1704960000000&signature=01l0U7nCxzkknY-KQ0rQabOsdH3HVhDu8e3lA1cBMGE&downloadName=%ED%94%84%EB%A6%AC%EC%A0%A0%ED%85%8C%EC%9D%B4%EC%85%9814.png)

### 팀 페이지 디자이닝

![팀 페이지 디자이닝](https://file.notion.so/f/f/aaedcf79-6b31-4898-9a1f-5e2ad8ae925e/e7925170-bd44-4099-8360-8a767d29c407/%ED%94%84%EB%A6%AC%EC%A0%A0%ED%85%8C%EC%9D%B4%EC%85%9813.png?id=7517ebbd-d7cb-4992-af1f-7f6e623257e3&table=block&spaceId=aaedcf79-6b31-4898-9a1f-5e2ad8ae925e&expirationTimestamp=1704960000000&signature=f0LAF0CeWRpJ7CEwgDg562kWvFzfIU0EiZjaVDVTkSY&downloadName=%ED%94%84%EB%A6%AC%EC%A0%A0%ED%85%8C%EC%9D%B4%EC%85%9813.png)

앞으로도 저희 BDD를 많이 사랑해주세요.

감사합니다.
~~~

![최종 이메일 발송 이미지](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/bdd/mail/quickStart/step6.png?raw=true)

## Thymeleaf를 이용한 관리자 페이지 구현

매번 포스트 맨으로 '\n'으로 개행을 넣어 이메일을 발송할 수 없으니 타임리프를 사용해서 관리자 전용 페이지를 간단하게 만들었습니다. 또한, 선배님들의 이메일 추가, 삭제, 수정, 발송되기 전 메일 미리보기 페이지를 만들었습니다.

자세한 UI와 사용 방법은 [BDD-CLUB Spring bulk mailing Service README](https://github.com/BDD-CLUB/bulk-mailing-service)에서 확인하실 수 있습니다.

## 결론

JavaMailSender를 통해서 HTML, 텍스트, 이미지를 발송할 수 있습니다. 이 서비스에서는 "우아한테크 뉴스레터"의 HTML 폼을 사용했지만 프론트 분들의 도움을 받아 BDD만의 커스텀 HTML 파일을 파싱해 메일을 보낼 수 있습니다. 그리고, 정규표현식을 통해 마크다운 문법을 지원할 수 있었습니다.

위의 과정들을 통해 쉽게 볼드체와 이미지를 첨부할 수 있게 되었고, 포맷팅 또한 깔끔해졌습니다. 하지만 실무에서는 배치 처리를 할 때 가장 신경쓰는 부분은 성능입니다. 위의 서비스에서는 하나의 메일을 보낼 때 마다 4초의 시간이 걸립니다. 이럴 경우에 회원이 1만명이라면 11시간이 걸리고 10만명이라면 111시간이 걸리게 됩니다.

다음 글에서는 어떻게 배치처리를 할 때 성능을 개선할 수 있는지 알아보겠습니다.
