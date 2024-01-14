---
published: false
title:  "Spring Batch - BDD 내부 메일링 서비스 개발 (3) - AsyncItemProcessor 성능 튜닝"
categories:
  - spring
---

## 서론

[Spring Batch - BDD 내부 메일링 서비스 개발 (2) - ItemReader 성능 튜닝](https://02ggang9.github.io/spring/BuilMailing2/) 포스팅에서 JpaPagingItemReader에서 JdbcCursorItemReader로 변경해 약 12배의 성능 개선을 이루었습니다. 하지만 메일을 보내는 API가 4초 걸리기 때문에 회원이 10,000명 이라면 40,000초(11시간)가 걸리게 됩니다. 메일 발송하는 processor에서 싱글 스레드로 할 경우 고정적으로 11시간이 걸리게 되므로 멀티 스레드로 메일 API를 호출하도록 변경했습니다.

이번 글에서는 AsyncItemProcessor에 대해서 정말 간단하게 알아보고 비동기 적용 후 얼마나 성능이 개선되었는지 알아보도록 하겠습니다. 자세한 설명은 다른 포스팅에서 따로 작성하겠습니다.

## AyncItemProcessor

AyncItemProcessor는 단지 동기적으로 처리하는 ItemProcessor에게 별도의 스레드를 할당하고 작업을 위임하는 역할을 합니다. 따라서 AyncItemProcessor에게 동기적으로 처리되고 있는 ItemProcessor를 인자 값으로 넘겨주기만 하면 됩니다. Spring Batch 내부적으로 동기화 이슈를 잘 해결했기 때문에 크게 걱정할 필요는 없습니다. 다만, 커스텀으로 AyncItemProcessor를 제작한다면 고려해야 할 상황이 많아 되도록이면 제공하고 있는 구현체를 사용하는 것을 추천합니다.

~~~java
@Bean
public Step sendMailStep(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager) {
    return new StepBuilder("sendMailStep", jobRepository)
            .<Member, Member>chunk(1000, platformTransactionManager)
            .listener(stopWatchJobListener)
            .reader(mailItemReader())
            .processor(mailAsyncItemProcessor())
            .writer(mailAsyncItemWriter())
            .faultTolerant()
            .retryLimit(5)
            .retry(MailException.class)
            .build();
}

@Bean
public AsyncItemProcessor mailAsyncItemProcessor(
) {
    AsyncItemProcessor<Member, Member> asyncItemProcessor = new AsyncItemProcessor<>();
    asyncItemProcessor.setDelegate(mailItemProcessor(null, null)); // 단지 위임만 함.
    asyncItemProcessor.setTaskExecutor(new SimpleAsyncTaskExecutor()); // 스레드 실행

    return asyncItemProcessor;
}
~~~

## 최종 성능 측정







## 결론


