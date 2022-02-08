---
title : "[Spring] TestRestTemplate Patch 요청 오류날 때"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
# 문제
게시글 수정 테스트 코드를 작성하던 중, restTemplate의 exchange() 메서드에 아래와 같이
```java
HttpEntity<PostsUpdateRequestDto> requestEntity = new HttpEntity<>(requestDto);


ResponseEntity<Long> responseEntity = restTemplate.exchange(url, HttpMethod.PATCH, requestEntity, Long.class);
```
PATCH 요청을 보내면 오류가 계속 떴는데, PutMapper로 바꾸고 exchange에도 PUT 요청을 보내면 테스트가 통과되었다. 그래서 PATCH로 했을 때의 로그를 확인해 보니,

org.springframework.web.client.ResourceAccessException: I/O error on PATCH request for "http://localhost:65324/api/v1/posts/1": Invalid HTTP method: PATCH; nested exception is java.net.ProtocolException: Invalid HTTP method: PATCH

PATCH를 아예 올바른 HTTP 요청으로 인식하지 못하는 것 같았다. 그래서 구글링을 해보니 스택오버플로우에 해결방법이 나와있어 문제를 해결할 수 있었다.

[출처링크](https://stackoverflow.com/questions/29447382/resttemplate-patch-request)

따로 apache HttpClientFactory를 주입하여 RestTemplate을 생성해주면 된다.

apache를 사용하기 위해 build.gradle 파일에 의존성을 아래와 같이 추가해주고,

```java
implementation 'org.apache.httpcomponents:httpclient:4.5.13'
```

테스트 코드가 동작하기전에 준비할 작업을 모아두는 @Before 어노테이션의 setUp 메서드에 다음을 추가해주면 된다.

```java
@Before
public void setup() {
    restTemplate.getRestTemplate().setRequestFactory(new HttpComponentsClientHttpRequestFactory());
}
```

이렇게 하고 다시 PATCH 요청을 exchange를 사용해서 보내주면 올바르게 요청이 보내져 테스트가 통과하는것을 볼 수 있다.
