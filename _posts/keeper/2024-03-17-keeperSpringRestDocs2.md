---
published: true
title:  "KEEPER - KEEPER Spring Rest Docs DSL 제작기 (2)"
categories:
  - keeper
---

## 서론

저번 주에 KEEPER Spring Rest Docs DSL v0.0.1을 만들었는데 한계점에 봉착했었습니다. 한정적인 HTTP METHOD 지원, 알아보기 힘든 코드, 조건 분기와 기능 확장의 어려움이 있었습니다. 앞으로의 유지 보수를 생각했을 때, 한번 코드를 갈아엎어야겠다고 생각했습니다.

Spring Rest Docs가 어떤 구조로 되어있는지 구조도를 그려보고, 어떤 타입과 인터페이스를 사용하는지를 공부했습니다. 밑에서는 기존 코드의 문제점과 Spring Rest Docs의 구조와 개선 후 코드에 대해서 알아보도록 하겠습니다.

현재 기능이 안정되었다고 생각해서 PR을 올렸고 [Feature/#413 rest docs를 쉽게 작성할 수 있도록 한다](https://github.com/KEEPER31337/Homepage-Back-R2/pull/431)에서 코드를 확인하실 수 있습니다.

## 기존 코드의 문제점

### 하나의 클래스에 모든 메서드가 들어가는 문제

KEEPER DSL을 사용할 때 코드의 depth가 깊어지는 것이 싫어서 하나의 클래스에 모든 함수를 정의했습니다.

~~~java
class RestDocs(
        private val mockMvc: MockMvc,
        private val documentName: String?,
        private val method: HttpMethod,
        private val uri: String,
        private val pathParams: Array<*>?,
) {
    ...
    private val pathVariables = mutableListOf<Field>()
    private val cookieVariables = mutableListOf<Cookie>()
    ...

    fun expect(httpStatus: HttpStatus, contentType: MediaType) {
        ...
    }

    fun pathVariable(vararg fields: Field) {
        pathVariables.addAll(fields)
    }

    fun cookie(vararg cookies: Cookie) {
        cookieVariables.addAll(cookies)
    }

    fun cookieVariable(vararg fields: Field) {
        cookieFields.addAll(fields)
    }

    fun requestJson(jsonBody: String) {
        this.requestJson = jsonBody
    }

    fun requestBody(vararg fields: Field) {
        requestFields.addAll(fields)
    }

    fun responseBody(vararg fields: Field) {
        responseFields.addAll(fields)
    }

    fun responseBodyWithPaging(vararg fields: Field) {
        responseFields.addAll(fields)
        ...
    }
}
~~~

하나의 클래스에 모든 함수를 집어넣었을 때 KEEPER DSL의 이점은 코드의 길이가 줄어든다는 점입니다. 저번 [KEEPER - KEEPER Spring Rest Docs DSL 제작기 (1)](https://02ggang9.github.io/keeper/keeperSPringRestDocs1/) 글을 봤을 때 12 Line이 줄어들었습니다. 

이때 당시에는 코드의 길이가 줄어들어서 간결해졌다고 생각했지만 완성하고 다시 보니 cookie()나 cookieVariable()과 같은 메서드가 너무 헷갈렸습니다. cookie() 메서드가 request에 담긴 cookie 값이고, cookieVariable은 response 명세입니다. KEEPER DSL을 사용하는 개발자 분들이 사용하기에 불편함을 느끼실 것 같았습니다.

또, 파싱하는 함수의 길이가 너무 길어져 유지보수성이 매우 떨어졌습니다. 다양한 필드 값들을 파싱하는 과정의 코드를 볼 때 눈살이 찌푸려졌습니다.

### 기능 확장의 어려움

MULTIPART 메서드를 지원하려고 했는데 MockMvc를 생성하는 기반 코드가 request 메서드를 사용했습니다. 

~~~java
// Spring Rest Docs 
public static MockHttpServletRequestBuilder request(HttpMethod httpMethod, String urlTemplate, Object... urlVariables) {
    return MockMvcRequestBuilders.request(httpMethod, urlTemplate, urlVariables).requestAttr("org.springframework.restdocs.urlTemplate", urlTemplate);
  }

public static MockMultipartHttpServletRequestBuilder multipart(String urlTemplate, Object... urlVariables) {
    return (MockMultipartHttpServletRequestBuilder)MockMvcRequestBuilders.multipart(urlTemplate, urlVariables).requestAttr("org.springframework.restdocs.urlTemplate", urlTemplate);
  }

public static MockMultipartHttpServletRequestBuilder multipart(String urlTemplate, Object... urlVariables) {
    return (MockMultipartHttpServletRequestBuilder)MockMvcRequestBuilders.multipart(urlTemplate, urlVariables).requestAttr("org.springframework.restdocs.urlTemplate", urlTemplate);
  }

// KEEPER DSL 파싱 코드
val result = mockMvc.perform(RestDocumentationRequestBuilders.request(method, uri, pathParams?.joinToString(", "))
                .contentType(MediaType.APPLICATION_JSON)
~~~

request 메서드를 사용해서는 multipart 메서드를 지원할 수 없었습니다. 기반 코드부터 잘못되서 더이상의 기능 확장이 어려워졌습니다. 이 때문에 Spring Rest Docs는 어떻게 문서를 파싱하는지 공부하게 되었습니다.

### IDE 메서드 추측(?) 기능 문제

하나의 클래스에 모든 메서드를 정의하니까 IDE가 10개 이상의 메서드(cookie, cookieVariable, requestJson, responseBody 등등)를 추측해 줬습니다. Spring Rest Docs처럼 체이닝을 걸 때 빌더 스코프에 맞는 메서드만 추측하는 것을 원했습니다.

## Spring Rest Docs 구조

### MockHttpServletRequestBuilder

처음 Rest Docs를 작성할 때 RestDocumentationRequestBuilders 클래스에 있는 메서드를 이용합니다. 이 메서드를 이용하면 최종적으로 MockHttpServletRequestBuilder 생성자를 호출하고 빌더 패턴으로 프로퍼티를 초기화 할 준비를 합니다.

~~~java
mockMvc.perform(get("/merits"))
mockMvc.perform(multipart("/merits"))

// RestDocumentationReqeustBuilders Class
public static MockHttpServletRequestBuilder get(String urlTemplate, Object... urlVariables) {
    return MockMvcRequestBuilders.get(urlTemplate, urlVariables).requestAttr("org.springframework.restdocs.urlTemplate", urlTemplate);
  }

// MockMvcRequestBuilders Class Return
public static MockHttpServletRequestBuilder get(String urlTemplate, Object... uriVariables) {
    return new MockHttpServletRequestBuilder(HttpMethod.GET, urlTemplate, uriVariables);
  }

public class MockHttpServletRequestBuilder implements ConfigurableSmartRequestBuilder<MockHttpServletRequestBuilder>, Mergeable {
  private final String method;
  private final URI url;
  private String contextPath;
  private String servletPath;
  ...

  methods...
}
~~~

KEEPER DSL의 docs 함수는 필수적으로 mockMvc, HTTP METHOD, URL을 입력받습니다. 두 번째 인자인 HTTP METHOD에 따라서 MockMvcRequestBuidlers의 메서드를 호출하면 될 것 같습니다.

~~~java
// docs Function
fun docs(
        mockMvc: MockMvc,
        method: DocsMethod, // Here !
        url: String,
        vararg pathParams: String,
        init: RestDocsResult.() -> Unit
): RestDocsRequestBuilder {
    val restDocsRequestBuilder = RestDocsRequestBuilder(mockMvc, method, url, pathParams) // Here !
    val requestBuilder = restDocsRequestBuilder.build()!!

    ...
}

// RestDocsRequestBuilder
class RestDocsRequestBuilder(
        private val mockMvc: MockMvc,
        private val method: DocsMethod,
        private val url: String,
        private val pathParams: Array<*>? = null
) {

    fun build(): MockHttpServletRequestBuilder? {
        val mockRequestBuilder = when (method) {
            DocsMethod.GET -> RestDocumentationRequestBuilders.get(url, pathParams?.joinToString(", "))
            DocsMethod.PUT -> RestDocumentationRequestBuilders.put(url, pathParams?.joinToString(", "))
            DocsMethod.POST -> RestDocumentationRequestBuilders.post(url, pathParams?.joinToString(", "))
            DocsMethod.DELETE -> RestDocumentationRequestBuilders.delete(url, pathParams?.joinToString(", "))
            DocsMethod.PATCH -> RestDocumentationRequestBuilders.patch(url, pathParams?.joinToString(", "))
            DocsMethod.MULTIPART -> RestDocumentationRequestBuilders.multipart(url, pathParams?.joinToString(", "))
        }

        return mockRequestBuilder
    }
}
~~~

그 후 빌더 패턴으로 메서드 체이닝을 걸면서 프로퍼티 값들을 수정하고 있습니다. 

~~~java
public MockHttpServletRequestBuilder content(byte[] content) {
    this.content = content;
    return this;
  }

public MockHttpServletRequestBuilder contentType(String contentType) {
    Assert.notNull(contentType, "'contentType' must not be null");
    this.contentType = contentType;
    return this;
  }
~~~

KEEPER DSL도 마찬가지로 DocsRequesstBuilder 클래스를 만들고 content 등 함수로 프로퍼티들을 수정하도록 하겠습니다. 프로퍼티의 타입은 Spring Rest Docs와 동일하게 했습니다.

~~~java
class DocsRequestBuilder {

    private val cookies: MutableList<Cookie> = mutableListOf()
    private var content: String? = null
    private var contentType: String? = null
    private var paramMap: MutableMap<String, String> = mutableMapOf()

    fun content(content: String) {
        this.content = content
    }

    fun contentType(contentType: MediaType) {
        this.contentType = contentType.toString()
    }

    fun cookie(vararg cookies: Cookie?) {
        Assert.notNull(cookies, "Cookies must not be null")
        cookies.filterNotNull().forEach { this.cookies.add(it) }
    }
}
~~~

### ResultActions

두 번째는 ResultMatcher 타입을 인자로 받는 andExpect 함수입니다. KEEPER DSL은 ResultMatcher 타입을 프로퍼티에 저장한 다음 체이닝을 걸 때 값을 가져오는 형식입니다.

~~~java
class DocsResultBuilder {

    private val resultMatcher: MutableList<ResultMatcher> = mutableListOf()

    fun expect(result: ResultMatcher) {
        resultMatcher.add(result)
    }

    fun build(mockMvc: ResultActions): ResultActions {
        resultMatcher.forEach { mockMvc.andExpect(it) }
        return mockMvc
    }
}
~~~

![image2](https://github.com/02ggang9/02ggang9.github.io/blob/master/_posts/images/keeper/restdocs/restdocs2.png?raw=true)


### Snippets

마지막으로는 andDo() 부분의 document 메서드인데 String 타입의 documentName과 Snippet 타입을 가변인자로 받습니다.

~~~java
mockMvc.perform(get("/merits")
              .cookie(new Cookie(ACCESS_TOKEN.getTokenName(), adminAccessToken)))
          .andExpect(status().isOk())
          .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
          .andDo(document("search-meritLog", // HERE !!
              requestCookies(
                  cookieWithName(ACCESS_TOKEN.getTokenName())
                      .description("ACCESS TOKEN %s".formatted(securedValue))
              ),
              responseFields(
                  pageHelper(getMeritLogResponse())
              )));

// MockMvcRestDocumentation Class 
public static RestDocumentationResultHandler document(String identifier, Snippet... snippets) {
    return new RestDocumentationResultHandler(new RestDocumentationGenerator(identifier, REQUEST_CONVERTER, RESPONSE_CONVERTER, snippets));
  }
~~~

KEEPER DSL은 위와 마찬가지로 cookieField, responseFields 등 프로퍼티에 저장한 다음 파싱할 때 스니펫으로 변환 후 체이닝을 걸고 있습니다. documentName은 어노테이션과 리플렉션을 활용하고 있습니다.

~~~java
class DocsResponseBuilder {

    private val responseFields: MutableList<Field> = mutableListOf()
    private val requestFields: MutableList<Field> = mutableListOf()
    private val cookieFields: MutableList<Field> = mutableListOf()
    private val pathFields: MutableList<Field> = mutableListOf()
    private var pageable: Boolean = false

    fun cookie(vararg cookies: Field) {
        cookieFields.addAll(cookies)
    }

    fun responseBody(vararg fields: Field) {
        responseFields.addAll(fields)
    }

    fun build(mockMvc: ResultActions) {
    val currentStackTraceElement = Thread.currentThread().stackTrace.firstOrNull { it.fileName?.endsWith("Test.kt") == true }
    val className = currentStackTraceElement?.className
    val methodName = currentStackTraceElement?.methodName

    val method = Class.forName(className).methods.firstOrNull { it.name == methodName }
    val documentName = method?.getAnnotation(Documentation::class.java)?.documentName

    mockMvc.andDo(
            MockMvcRestDocumentation.document(documentName, *snippets.toTypedArray())
        )
    }
}
~~~

## 개선 후 코드

~~~java
    // Spring Rest Docs
    @Test
    @DisplayName("상벌점 목록 조회를 성공해야 한다.")
    void 상벌점_목록_조회를_성공해야_한다() throws Exception {
      String securedValue = getSecuredValue(MeritController.class, "searchMeritLogList");

      mockMvc.perform(get("/merits")
              .cookie(new Cookie(ACCESS_TOKEN.getTokenName(), adminAccessToken)))
          .andExpect(status().isOk())
          .andExpect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
          .andDo(document("search-meritLog",
              requestCookies(
                  cookieWithName(ACCESS_TOKEN.getTokenName())
                      .description("ACCESS TOKEN %s".formatted(securedValue))
              ),
              responseFields(
                  pageHelper(getMeritLogResponse())
              )));
    }

    public class MeritApiTestHelper extends IntegrationTest {

    FieldDescriptor[] getMeritLogResponse() {
        return new FieldDescriptor[]{
            fieldWithPath("id").description("상벌점 로그의 ID"),
            fieldWithPath("giveTime").description("상벌점 로그의 생성시간"),
            fieldWithPath("awarderName").description("수상자의 이름"),
            fieldWithPath("awarderGeneration").description("수상자의 학번"),
            fieldWithPath("score").description("상벌점 점수"),
            fieldWithPath("meritTypeId").description("상벌점 타입의 ID"),
            fieldWithPath("reason").description("상벌점의 사유"),
            fieldWithPath("isMerit").description("상벌점 타입")
            };
        }
    }

    // KEEPER DSL
    @Documentation("search-meritLog-kt")
    fun `상벌점 목록 조회를 성공해야 한다`() {
        docs(mockMvc, DocsMethod.GET, "/merits") { 
            request {
                cookie(*memberTestHelper.getTokenCookies(admin))
            }

            result {
                expect(status().isOk())
                expect(content().contentType(MediaType.APPLICATION_JSON_UTF8))
            }

            response { 
                cookie(
                        ACCESS_TOKEN.tokenName means "ACCESS TOKEN",
                        REFRESH_TOKEN.tokenName means "REFRESH TOKEN",
                )

                responseBodyWithPaging(
                        "content[].id" means "상벌점 로그의 ID",
                        "content[].giveTime" means "상벌점 로그의 생성시간",
                        "content[].score" means "상벌점 점수",
                        "content[].meritTypeId" means "상벌점 타입의 ID",
                        "content[].reason" means "상벌점 사유",
                        "content[].isMerit" means "상벌점 타입",
                        "content[].awarderName" means "수상자의 이름",
                        "content[].awarderGeneration" means "수상자의 학번",
                )
            }
        }
    }
~~~

## 결론
KEEPER DSL v0.0.1은 기능의 확장에 막혀 있었습니다. 하나의 클래스에 모든 메서드가 들어가 있어 메서드 명이 헷갈리고 IDE 자동완성 기능의 혜택을 받을 수 없었습니다. 

수정한 KEEPER DSL은 Spring Rest Docs의 빌딩 과정을 따라 했기 때문에 기능 확장에 어려움이 없습니다. 또, DocsRequestBuilder, DocsResultBuilder, DocsResponseBuilder 클래스에 분할해서 메서드를 정의했기 때문에 IDE 자동완성 기능을 적극적으로 사용할 수 있습니다.

기존 버전에 비해서 코드의 길이는 길어졌기만 가독성이 눈에 띄게 높아졌습니다. 또 docs 를 작성할 때 "이런 request를 보내면 result가 이럴 것이라고 예상하고 response는 이렇게 올 것이다"라는 자연스러운 흐름으로 코드를 작성할 수 있습니다.
