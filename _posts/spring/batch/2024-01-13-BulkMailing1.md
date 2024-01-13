---
published: true
title:  "Spring Batch - BDD 내부 메일링 서비스 개발 (1) - JavaMailSender를 통한 메일 발송과 마크다운 문법 지원, 메일 포맷팅"
categories:
  - spring
---

## 서론

제가 속해 있는 BDD(부산 개발자 소모임)는 명예회원이라는 제도가 있습니다. 명예회원은 실무에 종사하고 계시는 선배님들이고, 세미나 발표와 커피챗, 슬랙에서의 정보 공유 활동을 BDD와 같이 하시고 계십니다. 명예회원님들께 한달에 한 번씩 메일로 BDD 활동 내역과 발생했던 일(뉴스레터)을 보내드려 지속해서 BDD에 관심을 가지실 수 있게 메일링 시스템을 만들어 보라고 한상곤 교수님께서 말씀을 해 주셨습니다. 말씀을 듣자마자 Spring Batch를 공부하고 간단한 메일링 프로그램을 만들어 봤습니다.

이번 포스트에서는 JavaMailSender를 통한 메일 발송과 간단한 마크다운 문법 지원, 포맷팅을 맞추기 위한 과정을 담았습니다. 다음 포스트에서는 성능 개선에 관한 내용으로 찾아뵙겠습니다.

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

[BDD 뉴스레터](https://github.com/BDD-CLUB/bulk-mailing-service/blob/e1fa6e688f3462f7d000ad1394ac410383aa2a14/src/main/java/io/springbatch/springbatch/service/EmailService.java)

정규표현식을 사용해서 {} 중괄호 안에 있는 문자열을 "strong" 태그로 감싸도록 했습니다. 1차적으로 완성된 메일 서비스로 메일을 전송할 때 이미지는 아래와 같습니다.

![1차 뉴스레터](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/batch/블로그글/1차뉴스레터.png?raw=true)



과거 한상곤 교수님의 공학작문및발표 수업에서 사내에서 문서 변환 도구(예를 들어, 마크다운에서 PDF로)를 만들어 사용했다는 일화가 기억났습니다. 메일링 서비스에서 볼드체를 지원한다면 보다 깔끔한 본문을 작성할 수 있다고 생각했습니다. 또, 일반적인 메일은 양옆이 넓고 글자가 밋밋해 보였습니다.



## 마크다운 문법 제공 



## 결론


