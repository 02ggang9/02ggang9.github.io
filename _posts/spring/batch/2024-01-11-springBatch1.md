---
published: false
title:  "Spring - 연습장"
categories:
  - spring
---

STEP들은 스프링 부트가 뜰때 Simple Job의 속성에 저장됨. STEPS

Simple Job이 실행을 시킴

단하나의 STEP이 실패하면 잡이 실패함
따라서 맨 마지막에 실행한 STEP의 batchStatus가 job의 최종 BATCHSTATUS가 된다.

validator()
Job 실행에 꼭 필요한 잡 파라미터를 검증하는 용도
잡 파라미터는 키, 벨류 형태임.
DefaultJobParameterValidator 구현체를 지원하며, 좀 더 복잡한 제약 조건이 있으면 인터페이스를 직접 구현할 수 있음.

~~~java
    @Bean
    public Job mailJob(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager) {
        log.info("mailJob Execute");
        return new JobBuilder("mailJob", jobRepository)
                .start(sendMailStep(jobRepository, platformTransactionManager))
                .validator(new DefaultJobParametersValidator(new String[]{"mailSubject", "mailMessage"}, new String[]{"startTime", "randomStringForTest"}))
                .build();
    }
~~~

~~~java
    @Bean
    public Job mailJob(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager) {
        log.info("mailJob Execute");
        return new JobBuilder("mailJob", jobRepository)
                .start(sendMailStep(jobRepository, platformTransactionManager))
                .validator(parameters -> {
                    if (parameters.getString("mailSubject") == null || parameters.getString("mailMessage") == null) {
                        throw new JobParametersInvalidException("Mail Job의 필수 필수 파라미터 값이 없습니다");
                    }
                })
                .build();
    }
~~~

~~~java
    @Bean
    public Job mailJob(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager) {
        log.info("mailJob Execute");
        return new JobBuilder("mailJob", jobRepository)
                .start(sendMailStep(jobRepository, platformTransactionManager))
                .validator(parameters -> {
                    if (parameters != null && (parameters.getString("mailSubject") == null || parameters.getString("mailMessage") == null)) {
                        throw new JobParametersInvalidException("Mail Job의 필수 필수 파라미터 값이 없습니다");
                    }
                })
                .build();
    }
~~~

~~~java
    @Bean
    public Job mailJob(JobRepository jobRepository, PlatformTransactionManager platformTransactionManager) {
        log.info("mailJob Execute");
        return new JobBuilder("mailJob", jobRepository)
                .start(sendMailStep(jobRepository, platformTransactionManager))
                .validator(mailJobParameterValidator)
                .build();
    }
    @Component
public class MailJobParameterValidator implements JobParametersValidator {

    @Override
    public void validate(JobParameters parameters) throws JobParametersInvalidException {
        isParameterValid(parameters, "mailSubject");
        isParameterValid(parameters, "mailMessage");
    }

    private void isParameterValid(JobParameters parameters, String requiredKey) throws JobParametersInvalidException {
        if (parameters.getString(requiredKey) == null) {
            throw new JobParametersInvalidException("Mail Job의 필수 파라미터 값인 " + requiredKey + "가 없습니다");
        }
    }
}

~~~


ChunkProvider

ItemReader가 Item을 한 개씩 읽어서 리턴함.
읽을 아이템이 null 이면 반복문 종료 및 전체 chunk 프로세스 종료
