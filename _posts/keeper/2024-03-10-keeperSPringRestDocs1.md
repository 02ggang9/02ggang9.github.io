---
published: true
title:  "KEEPER - KEEPER Spring Rest Docs DSL 제작기 (1)"
categories:
  - keeper
---

## 서론

한달 전, 센디라는 스타트업 회사에 들어가고 싶어서 코틀린 스터디에 참여했습니다. 코틀린 언어를 공부할수록 정말 재밌고 좋은 언어라고 생각해서 코프링 해보자는 말을 주변 사람들에게 정말 많이 했습니다. 하지만 SigmaDream(닉네임) 교수님의 프로그래밍 언어론 수업에서 코프링을 사용하는 친구는 기절시키라고 하셔서 순간 이목이 집중되었습니다(주변에 BDD 사람이 정말 많았습니다..)

제가 코틀린을 공부하면서 정말 좋아보였던 것이 확장함수, 고차함수와 DSL 이였습니다. 이 세가지를 조합하면 코드가 깔끔해지고, 재사용성도 좋아져 사용할 곳이 없나 찾아보다가 [Kotlin으로 DSL 만들기: 반복적이고 지루한 REST Docs 벗어나기](https://toss.tech/article/kotlin-dsl-restdocs)라는 기술문서를 발견했습니다. 이 글은 Spring Rest Docs 코드를 작성할 때, 반복적인 코드 호출과 코드의 가독성이 떨어져서 토스 자체 Spring Rest Docs DSL을 만들었다고 합니다. 저도 Spring Rest Docs를 작성할 때 반복되는 코드, 중괄호와 쉼표 사용때문에 가독성이 정말 떨어진다고 생각했습니다. 또, 최근 KEEPER 프로젝트에 신규 회원분들이 참여하셨는데 Spring Rest Docs 작성에 어려움을 겪고 계셨습니다. 이런 반복되는 코드 작성과 신규 회원 분들의 학습 비용을 줄이기 위해서 KEEPER Spring Rest Docs DSL을 만들기 시작했습니다.

아래의 본문에서는 Kotlin DSL을 사용할 때 주로 사용되는 문법, 발생한 에러와 1차 구현된 코드를 살펴보도록 하겠습니다. 코드는 [KEEPER R2 Github](https://github.com/KEEPER31337/Homepage-Back-R2/blob/Feature/%23413-RestDocs%EB%A5%BC_%EC%89%BD%EA%B2%8C_%EC%9E%91%EC%84%B1%ED%95%A0_%EC%88%98_%EC%9E%88%EB%8F%84%EB%A1%9D_%ED%95%9C%EB%8B%A4/src/test/java/com/keeper/homepage/global/docs/util.kt)에서 확인하실 수 있습니다.

## 중위 함수

## 확장 함수

## 발생한 에러와 트러블 슈팅

## KEEPER Spring Rest Docs DSL vs Spring Rest Docs

아래는 각각 KEEPER Spring Rest Docs DSL(이하 KEEPER Docs)과 기존의 Spring Rest Docs를 사용할 때의 코드입니다. KEEPER Docs를 사용하면 중위 함수 사용으로 자연어처럼 코드를 작성할 수 있고, 가독성이 올라갑니다. 반면에 Spring Rest Docs는 수많은 소괄호와 쉼표로 인해 전에 작성한 코드를 살펴보면서 코드를 작성해야 합니다. 저는 과거 코드를 보면서 하더라도 실수를 많이 했습니다.

아직 multipart 등등 많은 기능이 지원되고 있지 않은데, 빠른 시일 내에 만들도록 하겠습니다. 

~~~java
// KEEPER Spring Rest Docs DSL -> 16 Line + 가시성
@Test
fun `상벌점 조회는 성공해야 한다`() {
    restDocs(mockMvc, "search-meritType", HttpMethod.GET, "/merits/types") {
        expect(HttpStatus.OK, MediaType.APPLICATION_JSON_UTF8)
        cookie(*memberTestHelper.getTokenCookies(admin))
        cookieVariable(
                ACCESS_TOKEN.tokenName means "ACCESS TOKEN",
                REFRESH_TOKEN.tokenName means "REFRESH TOKEN",
        )
        responseBodyWithPaging(
                "content[].id" means "상벌점 타입의 ID",
                "content[].score" means "상벌점 점수",
                "content[].detail" means "상벌점 사유",
                "content[].isMerit" means "상벌점 타입",
        )
    }
}
~~~

~~~java
// Spring Rest Docs -> 28 Line + 가시성 x
@Test
@DisplayName("상벌점 타입 조회를 성공해야 한다.")
void 상벌점_조회는_성공해야_한다() throws Exception {
    String securedValue = getSecuredValue(MeritController.class, "searchMeritType");
    mockMvc.perform(get("/merits/types")
            .cookie(new Cookie(ACCESS_TOKEN.getTokenName(), adminAccessToken)))
        .andExpect(status().isOk())
        .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
        .andDo(document("search-meritType",
            requestCookies(
                cookieWithName(ACCESS_TOKEN.getTokenName())
                    .description("ACCESS TOKEN %s".formatted(securedValue))
            ),
            responseFields(
                pageHelper(getMeritTypeResponse())
            )));
}

public class MeritApiTestHelper extends IntegrationTest {
    FieldDescriptor[] getMeritTypeResponse() {
        return new FieldDescriptor[]{
            fieldWithPath("id").description("상벌점 타입의 ID"),
            fieldWithPath("score").description("상벌점 점수"),
            fieldWithPath("detail").description("상벌점의 사유"),
            fieldWithPath("isMerit").description("상벌점 타입")
        };
    }
}
~~~

![image1](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/restdocs/restdocs1.png?raw=true)

## 어노테이션과 리플랙션을 활용하도록 수정

~~~java
@Documentation("search-meritType-kt")
fun `상벌점 조회는 성공해야 한다`() {
    restDocs(mockMvc, HttpMethod.GET, "/merits/types") {
        expect(HttpStatus.OK, MediaType.APPLICATION_JSON_UTF8)
        cookie(*memberTestHelper.getTokenCookies(admin))
        cookieVariable(
                JwtType.ACCESS_TOKEN.tokenName means "ACCESS TOKEN",
                JwtType.REFRESH_TOKEN.tokenName means "REFRESH TOKEN",
        )
        responseBodyWithPaging(
                "content[].id" means "상벌점 타입의 ID",
                "content[].score" means "상벌점 점수",
                "content[].detail" means "상벌점 사유",
                "content[].isMerit" means "상벌점 타입",
        )
            }       
        }
~~~

## 결론