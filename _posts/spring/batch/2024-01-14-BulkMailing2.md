---
published: true
title:  "Spring Batch - BDD 내부 메일링 서비스 개발 (2) - ItemReader 성능 튜닝"
categories:
  - spring
---

## 서론

[BDD 내부 메일링 서비스 개발 (1)](https://02ggang9.github.io/spring/BulkMailing1/)에서 전체적인 UI와 마크다운 문법을 지원함으로써 손쉽게 메일을 보낼 수 있었습니다. 하지만 회원 1명에게 보낼 때 마다 걸리는 시간이 4s 걸렸습니다. 100명 기준으로 400s(약 7분), 1만명 기준으로 40000s(약 12시간)이 걸리게 됩니다. 

사태의 심각성을 느끼고 성능 튜닝을 하기위해서 여러 자료를 찾았습니다. 이번 글에서는 Batch에서 item을 읽어오는 방법과 메일링 서비스가 느린 이유, 개선 방법에 대해서 알아보겠습니다.

## Chunk Processing

배치 처리가 필요한 이유는 대규모의 데이터를 일괄적으로 처리할 수 없기 때문입니다. 데이터가 1000개일 경우에는 충분히 한번에 처리할 수 있지만 1000만개의 데이터일 경우에는 불가능합니다. 따라서 데이터를 1000개 단위로 나누고, 1만번 처리해야 합니다.

데이터를 나누는 대표적인 방법으로 Pagination이 있습니다. Spring Batch에서는 JpaPagingItemReader, JdbcPagingItemReader 등 다양한 구현체를 지원합니다. BDD 메일링 서비스는 JpaPagingItemReader를 사용했습니다.

## JpaPagingItemReader가 느린 이유

Paging 기법은 page number와 page size로 데이터를 잘라오는 방식입니다. MySQL에서는 limit와 offset 구문을 사용하는데 이 방법은 뒤로 갈수록 Item 조회가 굉장히 느립니다.

이유는 [Yun 님의 블로그](https://cheese10yun.github.io/spring-batch-reader-performance/)에서 찾을 수 있었습니다. 500백만 데이터 기준으로 대략 200백만 데이터까지는 인덱스를 통해 row 행에 접근할 수 있습니다. 하지만 그 이후에는 인덱스를 통해 row 행에 접근하지 않고 테이블의 모든 row를 스캔하고 버리는 형식의 전략을 취하고 있습니다. 이유는 아래의 Yun님의 글에서 알 수 있습니다.

> 인덱스를 통해 테이블의 레코드를 읽는 것은 인덱스를 거치지 않고 바로 테이블의 레코드를 읽는 것보다 높은 비용이 드는 작업입니다. 만약 100만 건의 레코드가 있다고 가정하고 그중 50만 건을 읽어야 하는 쿼리가 있다고 하면 테이블 전체를 읽어서 필요 없는 50만 건을 버리는 것이 효율적인지, 인덱스를 통해 필요한 50건만 읽어 오는 것이 효율적인지 옵티마이저는 판단해야 합니다. 일반적으로 DBMS의 옵티아미저에서 인덱스를 통해 레코드를 1건 읽는 것은 테이블에서 직접 레코드 1건을 읽는 것보다 4~5배 정도 비용이 발생한다고 예측합니다. 즉, 인덱스를 통해 읽어야 할 레코드의 건수(옵티마이저가 예측한)가 전체 테이블의 20~25%를 넘어서면 인덱스를 이용하지 않고 직접 테이블을 모두 읽어서 필요한 레코드만 필터링하는 방식이 효율적이라고 판단하게 됩니다.

![Yun님 블로그 사진 참조](https://raw.githubusercontent.com/cheese10yun/blog-sample/master/batch-study/docs/img/limit_3.png)

따라서 후반 청크에서는 데이터를 Full Scan 후 디스크로부터 데이터를 가져오기 때문에 필수적으로 느려질 수 밖에 없습니다.

## JdbcCursorItemReader 사용과 JpaCursorItemReader를 사용하면 안되는 이유

### JdbcCursorItemReader

Paging 말고 Cursor 방식으로도 데이터를 나눌 수 있습니다. Spring Batch에서는 JdbcCursorItemReader와 JpaCursorItemReader 등 다양한 구현체를 지원합니다. JdbcCursorItemReader는 ResultSetStreaming 방식으로 데이터를 한건 한건 조회합니다. 이렇게 할 경우에 일정한 속도로 데이터를 읽어올 수 있고 일정한 Heap 메모리를 사용할 수 있습니다.

![Yun님 블로그 Heap 사진 참조](https://raw.githubusercontent.com/cheese10yun/blog-sample/master/batch-study/docs/img/JdbcResultSetStreaming.png)

하지만 Cursor 방식은 데이터베이스와 애플리케이션 사이에 커넥션을 계속 물고있기 때문에 Timeout이 발생할 수 있습니다. 이를 해결하기 위해 default Timeout 설정을 길게 잡아줘야 합니다.

### JpaCursorItemReader

JpaCursorItemReader는 20,480,000번째 row를 조회할 때 20,480,000개의 모든 조회 데이터를 메모리에 올리기 때문에 OOM을 유발할 수 있습니다. 

![Yun님 블로그 Heap 사진 참조2](https://raw.githubusercontent.com/cheese10yun/blog-sample/master/batch-study/docs/img/JdbcResultSet.png)


## ItemReader 변경 후 성능 변화 측정

| ItemReader 종류        | DB 데이터 개수 | 걸린시간 |
|----------------------|-----------|------|
| JpaPagingItemReader  | 100,000   | 60s  |
| JdbcCursorItemReader | 100,000   | 5s   |

![JpaPagingItemReader](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/batch/성능개선/reader_paging1.png?raw=true)

![JdbcCursorItemReader](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/spring/batch/성능개선/reader_cursor1.png?raw=true)

DB row 개수가 100,000개 기준으로 성능이 약 12배 증가했습니다. Paging 특정상 데이터 수가 늘어날수록 성능 차이는 심할 것 이라고 생각합니다. 아래는 개선된 코드입니다. 자세한 코드는 [v1.0.1 commit](https://github.com/BDD-CLUB/bulk-mailing-service/blob/40ec5cd0d0da67c8562b9c7ac44768e3c01fc200/src/main/java/io/springbatch/springbatch/config/batch/mailJob/MailJobConfiguration.java) 에서 확인하실 수 있습니다.

~~~java
    @Bean
    public Step sendMailStep(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager) {
        return new StepBuilder("sendMailStep", jobRepository)
                .<Member, Member>chunk(1000, platformTransactionManager)
                .reader(mailItemReader())
                .writer(mailItemWriter())
                .faultTolerant()
                .retryLimit(5)
                .retry(MailException.class)
                .build();
    }

   @Bean
   public ItemReader<Member> mailItemReader() { // JpaPagingItemReader
       log.info("itemReader {}", LocalDateTime.now());

       return new JpaPagingItemReaderBuilder<Member>()
               .name("memberMailReader")
               .entityManagerFactory(this.entityManagerFactory)
               .pageSize(20)
               .queryString("SELECT m FROM Member m")
               .build();
   }

    @Bean
    public ItemReader<Member> mailItemReader() { // JdbcCursorItemReader
        log.info("itemReader {}", LocalDateTime.now());

        return new JdbcCursorItemReaderBuilder<Member>()
                .name("JdbcCursorItemReader")
                .fetchSize(1000)
                .sql("SELECT id, name, email FROM member")
                .rowMapper((rs, rowNum) -> {
                    return new Member(rs.getString(2), rs.getString(3));
                })
                .dataSource(dataSource)
                .build();
    }
~~~

## 한 발자국 더

위에서 JdbcCursorItemReader를 사용해서 엄청난 성능 개선을 이루었습니다. 하지만 위 코드의 문제점은 네이티브 쿼리를 작성해야 한다는 점입니다. 이를 해결하기 위해서 카카오 정산팀은 Query DLS + Exposed + 코틀린 조합을 사용해 QueryDlsZeroOffsetItemReader를 자체 제작해 사용하고 있습니다. JpaPagingItemReader의 문제점인 limit를 항상 0으로 설정해 성능이 느려지는 단점을 없앴고, Query DSL을 사용해 쿼리 작성 실수를 줄이고 있습니다.

현재 메일링 서비스는 읽을 데이터가 정말 단순해 네이티브 쿼리를 작성해도 무리가 없을것이라 생각했지만 실무에서는 카카오 정산팀처럼 자체 제작한 ItemReader를 사용하는 것이 좋겠다고 생각했습니다. 기회가 된다면 위의 3가지 조합을 통해 자체 라이브러리를 제작해보도록 하겠습니다.

## 결론

v1.0.0 버전의 메일링 서비스는 JpaPagingItemReader를 사용해 굉장히 느렸습니다. 느린 이유는 청크 후반부에는 Full Scan 전략을 사용하기 때문이였습니다. 이를 해결하기 위해서 일정한 속도로 데이터를 가져올 수 있는 Cursor 방식을 선택했고 적용 결과 성능은 12배 향상되었습니다. Spring Batch에서 제공하는 JdbcCursorItemReader 구현체를 사용해도 되지만 네이티브 쿼리 작성이라는 단점을 없애기 위해 Query DSL을 도입한 ItemReader를 자체 제작할 수 있습니다.

메일링 코드는 옆의 [BDD-CLUB/bulk-mailing-service Github](https://github.com/BDD-CLUB/bulk-mailing-service)에서 확인하실 수 있습니다.

## 참고자료

[[Data] Batch Performance 극한으로 끌어올리기: 1억 건 데이터 처리를 위한 노력 / if(kakao)dev2022](https://www.youtube.com/watch?v=2IIwQDIi3ys)

[[if kakao 2022] Batch Performance를 고려한 최선의 Reader](https://tech.kakaopay.com/post/ifkakao2022-batch-performance-read/)

[Spring Batch Reader 성능 분석 및 측정 part 1](https://cheese10yun.github.io/spring-batch-reader-performance/)

[Spring Batch Reader 성능 분석 및 측정 part 2](https://cheese10yun.github.io/spring-batch-reader-performance-2/)

[Spring Batch 가이드 - ItemReader](https://jojoldu.tistory.com/336)
