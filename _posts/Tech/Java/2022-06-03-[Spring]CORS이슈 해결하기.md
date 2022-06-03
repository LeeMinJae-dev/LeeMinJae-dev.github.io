---
title: "[Spring] CORS 설정하기"
categories: java
toc: true
toc_label: "On this page"
toc_sticky: true
---
## 서론
저는 지금 간단한 토이 프로젝트를 하나 진행중인데요.

프로젝트를 진행하면서 API를 대략 만들고 AWS에 배포를 완료했는데, 프론트엔드 개발자로부터 CORS를 해결해달라는 요청을 받았습니다.

그동안에는 그냥 혼자 웹페이지의 전부를 만들어봐서 해당 이슈에 대해 접해본적이 없기 때문에, 이러한 설정을 따로 해주어야하는지 전혀 몰랐습니다. 

그래서 이 게시물에서는 CORS는 무엇이고, 스프링에서 이러한 설정을 어떻게 해주어야 하는지 알아보도록 하겠습니다.

## SOP
사실 CORS에 대해 알기 위해서는 SOP에 대해 먼저 알아야합니다. 둘다 영어로 되어있어서 딱 듣고 알기는 어렵죠? **SOP**는 **Same-Origin Policy**의 준말입니다. 한글로 번역하면 **동일 출처 정책**이 되겠네요.

이는 어떤 출처(origin)에서 불러온 문서나 스크립트가 다른 출처에서 가져온 리소스와 상호작용을 하는것을 제한하는 보안방식입니다. 

여기서 **출처가 같다**는 것은, 두 URL의 **프로토콜, 호스트, 포트** 세개가 모두 같다는 것을 의미합니다. 

예를들어 http://domain/page1.html이 있다면, 

프로토콜은 http, 호스트는 domain, 포트는 http에서는 생략되어 보이지는 않지만 8080이겠죠. 

만약 이렇게 3개가 같은 주소에서 들어온 요청이 아니라면, SOP에 의해서 요청이 거절되는 것입니다.

왜 이런 보안방식을 사용할까요?

## SOP의 필요성
만약 SOP가 없다고 가정해봅시다.

여러분이 페이스북에 로그인을 하면 어떻게 되나요? 세션이 유지되기 때문에 일정 시간동안은 다시 로그인하지 않고도 페이스북에 로그인 된 상태일 것입니다. 

만약 이때 메일에서 어떤 해커가 페이스북에다가 "나는 바보입니다" 라고 게시물을 작성하는 자바스크립트 코드를 메일에 담아 여러분에게 전송했다면 어떻게 할까요? SOP가 없다면 여러분은 속수무책으로 이러한 공격에 당하기 쉽게 됩니다.

따라서 이를 막기위해서 SOP와 같은 정책으로, 우리의 정보를 더 안전하게 지키는 것이죠.

참고로 이런 SOP의 주체는 **웹브라우저**로, 여러분이 사용하는 크롬이나 엣지등의 웹 브라우저들이 해당 보안정책을 토대로 다른 출처의 요청을 제한합니다.

## CORS
자 이제 드디어 CORS가 등장합니다.

원래는 이러한 SOP에 의해서 무조건 같은 출처, **Sam-Origin**에서의 요청만 허락되었다고 배웠습니다. 하지만 개발을 하다보니, 오픈 API를 사용하기 위해서나, 여러가지 더 좋은 기능들의 개발을 위해 서로 다른 도메인간에도 통신이 필요한일이 생기게 되었습니다.

한마디로, 다른 출처, **Cross-Origin**의 요청도 처리할 필요성이 대두되기 시작한것이죠.

이를 위한 스펙이 바로 **CORS, Cross-Origin Resource Sharing**입니다. 

CORS는 웹브라우저에서 외부 도메인과 통신을 가능하게 하기위해서 만들어진 방식을 표준화 한 것으로, 서버와 클라이언트가 정해진 헤더를 통해 서로 요청에 응답할지 말지를 결정하게 됩니다.

## CORS의 동작 방식
CORS는 아래 3가지의 동작 방식을 갖습니다.

* 간단한 요청 (Simple Requests)
* 사전 요청 (Preflight Requests)
* 인증을 이용하는 요청 (Credential Requests)

### 간단한 요청
간단한 요청, Simple Requests는 기존 데이터를 손상시키지 않을 요청들에 의해서만 가능합니다. 

만약, DELETE나 PATCH요청과 같은 요청을 누구나 간단하게 요청해서 처리할 수 있다면, 요청하나만으로도 서버에 있는 데이터를 모두 지운다던가 이상하게 변경 할 수 있을 것입니다. 

따라서 Simple Requests는 아래의 조건을 만족할때만 가능합니다.

* GET, HEAD, POST중 한가지 요청일 것
* Custom Header가 존재하지 않을 것
* POST일 경우, Conent-Type이 application/x-www-form-urlencoded, mutipart/form-data. text/plain 중 하나일 것

사실상 요즘은 대부분 API요청을 application/json을 통해서 보내기 떄문에 Simple Requests는 잘 사용되지 않습니다.

예를들어 https://foo.example에서 서버로 Simple Requests로 요청을 보내게 되면, 

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```
이렇게 요청이 가게 됩니다.

Origin을 보면, 이 요청이 https://foo.example에서 왔음을 알 수 있죠. 

이 부분을 확인하고 서버에서는 이에 대한 응답으로 Access-Control-Allow-Origin 헤더를 다시 전송하게 됩니다.

예를들어 이러한 응답을 전송합니다.

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```
응답을 보면, 현재 Acces-Control-Allow-Origin이 *으로 설정되어 있기 때문에 모든 도메인에서 접근이 가능한 상태입니다. 따라서 https://foo.example외의 모든 도메인에서도 요청이 허용됩니다.

하지만 아래의 요청을 볼까요?

```
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin:  https://foo.example
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```
Acces-Control-Allow-Origin이 https://foo.example로 설정되어 있습니다. 이렇게 되면, https://foo.example의 도메인만 요청이 허락되는 것입니다. 따라서 이 도메인 외의 다른 도메인은 해당 리소스에 접근이 불가능 합니다. 

따라서, Origin 헤더에 전송된 값이 Access-Control-Allow-Origin 헤더에 포함되어 있을 때만, 해당 리소스에 대한 접근이 허용되는 것이죠.

### 사전 요청 (Preflight Requests)
사전요청은 본 요청을 보내기전에 사전 요청을 보내서 서버가 이에 대해서 응답이 가능한지를 확인하는 방법입니다. 

위의 간단한 요청외의 요청들은 서버에 있는 데이터를 손상시키거나 변조시킬 위험이 있는 코드를 보내는 것이 가능함으로, 사전요청으로 먼저 서버에게 해당 요청을 진행가능한지 확인 후, 해당 요청을 처리하게됩니다.

![image1](/assets/images/tech/Java/2022-06-03-[Spring]CORS이슈/image1.PNG)

위 그림을 보면, https://foo.example에서 예비요청을 보내고, 이에 대해서 Access-Control-Allow-Origin와 서버에서 현재 어떤것들을 허용하고 금지하는지에 대한 정보를 응답헤더에 담아 브라우저에게 다시 보내줍니다.

 여기서 서버가 허용하고 있는 요청만이 포함된 상태라면, 그제서야 본 요청을 서버에 보낼 수 있게 되는 것이죠.
 
 아래는 preflight요청의 예제입니다.
 ```
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: http://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type
```

밑에 세줄을 보면 Origin, Access-Control-Request-Method와 Access-Control-Request-Headers가 있는 것을 볼 수 있습니다. 

그럼 응답에 대한 예제를 봅시다.

```
HTTP/1.1 204 No Content
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
```

중간에 보면 Access-Control-Allow-Origin, 
Access-Control-Allow-Methods, Access-Control-Allow-Headers
가 있는 걸 볼 수 있습니다.

이들은 각각 서버 측 허가 출처, 허가 메서드, 허가 헤더, preflight 응답 캐시 기간인데요. 이러한 예비 요청을 통해서 위 네가지의 서버 허용 정책을 통과 해야만 다음 본요청을 보내는것이 가능한 것입니다.

따라서 Preflight 요청을 사용함으로써, 브라우저 단에서 미리 Allow-Origin에 없는 origin의 요청을 거절함으로써 서버를 안전하게 보호하는 것이죠.

### 인증을 이용하는 요청 (Credential Requests)
Credential Requests는 좀 더 보안을 강화시킨 요청으로, 인증 정보들을 추가로 쿠키등으로 요청에 담아서 보내는 방식입니다. 

서버는 이 사용자가 보낸 요청에 포함된 인증정보들로 한번 더 검증하여 인증이 된 사용자에게만 응답을 줍니다.

Credentila Requests에서는 Access-Control-Allow-Origin을 *로 설정할 수 없으며, 반드시 명시적인 URL들로 설정해주어야 합니다. 또, 반드시 응답 헤더에는 Access-Control-Allow-Credentails: true가 포함되어야만 Credentila Requests의 자격요건을 갖게됩니다.

그럼 이제 세가지 방식에 대해서도 알아봤으니, 스프링에서는 어떻게 하면 CORS를 서버에 적용하여 우리가 허락한 주소에만 응답을 줄 수 있는지 알아봅시다.

## 스프링에서 CORS 적용하기
스프링에서 CORS를 설정하는 방법은 크게 Annotation을 사용하는 방법과 Configuration을 사용하는 방법이 있습니다. 

## Annotation을 사용하여 설정
Controller  또는 메소드에 @CrossOrigin 어노테이션을 달아주면 됩니다.

```
@GetMapping("/v1/volunteers")
@CrossOrigin(origins = "*", allowHeaders = "*")
public List<VolunteerResponseDto> findAll() {
        return volunteerService.findAll();
   }
```
위 예시는 findAll()이라는 메소드에 CORS를 설정한 코드입니다. 

v1/volunteers로 요청이 들어오면, CrossOrigin에 설정해준 Origin에서 들어온 요청만 본응답을 받게 됩니다. 위 코드에서는 *로 와일드카드를 사용해주었으므로, 모든 출처에서의 요청에 대한 응답을 해주게 됩니다.

마찬가지로, 메소드뿐만 아니라 컨트롤러에도 적용할 수 있습니다.

```
@RequiredArgsConstructor
@RestController
@CrossOrigin(origins = "*")
public class VolunteerController {
    	private final VolunteerService volunteerService;

	@GetMapping("/v1/volunteers")
	public List<VolunteerResponseDto> findAll() {
	    return volunteerService.findAll();
	}
}
```

## Configuration을 사용하여 설정
스프링의 Configuration을 통해서 프로젝트 전체에 적용하는 방식입니다.

config 패키지에 WebConfig 클래스를 만들어준 뒤, WebMvcConfigurer를 implements 해줍니다.

```
@Configuration
public class WebConfig implements WebMvcConfigurer {    

}
```
여기서 CORS 설정을 위해서 필요한 메소드인 addCorsMappings 메소드를 오버라이딩 해줍니다.

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(final CorsRegistry registry) {
    }
}

```
이렇게 하면 CoreRegistry 클래스에 있는 addMapping을 이용해서 CORS를 적용해줄 URL 패턴을 정의 해줄 수 있습니다.

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(final CorsRegistry registry) {
        registry.addMapping("/**")

    }
}

```
현재는 /**로 와일드카드를 사용하여 모든 URL 패턴에 대해 허용해준 상태입니다. 

addMapping() default 값은 다음과 같습니다.

* Allow all origins.
* Allow "simple" methods GET, HEAD and POST.
* Allow all headers.
* Set max age to 1800 seconds (30 minutes).

이 옵션들은 addMapping() 과 마찬가지로 뒤에 추가해줌으로써 설정할 수 있습니다.

예를들어, 

```
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(final CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:8080")
                .allowedMethods("GET", "POST")
                .maxAge(3000);
    }
}

```
위 코드처럼 설정함으로써, Origin은 http://localhost:8080, 허용할 메서드는 GET, POST, 그리고 최대 캐싱 시간은 3000초로 설정할 수 있습니다.

제 프로젝트에서는 메소드에 따라 CORS의 origin을 다르게 할 필요는 없기 때문에, 전체에 적용가능한 Configuration 방식을 사용해서 CORS를 적용해주기로 했습니다.

이렇게 CORS에 대해 알아보고, 이를 스프링에서 어떻게 설정하는지 알아보았습니다. 




