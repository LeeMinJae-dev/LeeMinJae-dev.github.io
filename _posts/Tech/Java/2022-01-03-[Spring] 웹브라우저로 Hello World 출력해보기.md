---
title : "[Spring] 웹브라우저로 Hello World 출력해보기"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
이 게시물은 이동욱님의 "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"를 읽고 공부하며 작성한 게시물입니다.

## 시작하며...
어떤 프로그램이던 프레임워크던간에 새로운 기술을 시작했다면, 개발자들의 국룰을 Hello World를 띄워보는 것이다.       
Hello World를 띄우는것은 언제나 설레고 신난다! 그럼 스프링부트에서 웹 브라우저에 Hello World를 출력해보자..

## Application 클래스
먼저 메인으로 동작할 Application 클래스를 만들어주어야 한다. 

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### @SpringBootApplication
@SpringBootApplication 어노테이션은 이 클래스가 메인 클래스가 된다는것을 알려주는 어노테이션으로, @SpringbootApplication으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정하게 된다. 
특히나 항상 프로그램은 @SpringBootApplication이 있는 위치부터 설정을 읽어가기 떄문에 이 클래스는 항상 프로젝트 최상단에 위치하여야 한다. 

main 메소드에 있는 SpringApplication.run으로 인해서 내장 WAS (Web Application Server)를 실행한다. 내장 WAS란 별도로 외부에 WAS를 두지 않고 애플리케이션을 실행할 때 내부에서 WAS를 실행하는 것을 이야기 하는데, 이렇게 하면 항상 서버에 톰캣을 설치할 필요가 없게되고, 스프링 부트로 만들어진 jar 파일로 실행하면 된다.

스프링 부트는 내장 WAS를 사용하는데, 어플리케이션을 실행할 때 내부에서 WAS를 실행하기 때문에 외부에 별도의 추가 서버를 두지 않기 떄문에 서버를 배포할때 마다 일일히 WAS의 종류와 버젼, 설정을 일치시키지 않아도 된다.

여기서 WAS라는 개념이 등장하는데 이를 따로 정리해서 따로 게시물로 남겨놓았다..

> [WAS란 무엇일까?](https://learnote-dev.com/java/Java-WAS란-무엇일까/)

그럼 이제 Application클래스의 코드를 모두 작성했으니, 이제 Controller를 만들고 이를 테스트 해보자.

## HelloController
web이라는 이름을 가진 패키지를 만들어 그 안에 HelloController 클래스를 만들어 준 뒤, 아래와 같이 코드를 작성해 준다.

```
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import springboot.web.dto.HelloResponseDto;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name,
                                     @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}
```
### @RestController
- 컨트롤러를 JSON을 반환하는 컨트롤러로 만들어 준다.
- 예전에는 @ResponseBody를 각 메소드마다 선언했었는데, @RestController를 사용하여 쉽게 사용할 수 있게 되었다.

### @GetMapping
- HTTP Method인 Get의 요청을 받을 수 있는 API를 만들어주는 어노테이션이다.
- 예전에는 @RequestMapping으로 사용되었다. 이제 이 프로젝트는 /hello 요청이 오면 문자열 hello를 반환하는 기능을 가지게 되었다.

그렇다면 이렇게 작성한 코드가 제대로 작동하는지 알아보고, 웹API를 테스트하기위한 코드를 어떻게 작성하는 지를 배워보자.

## HelloControllerTest
```
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

import static org.hamcrest.Matchers.is;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(MockMvcRequestBuilders.get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}
```
### @RunWith(SpringRunner.class)
- 테스트를 진핼할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킨다.
- 여기서는 SpringRunner라는 스프링 실행자를 사용한다.
- 다시말해, 스프링 부트 테스트와 JUnit 사이에 연결자 역할을 한다.

### @WebMvcTest
- 여러 스프링 테스트 어노테이션 중, Web(Spring Mvc)에 집중할 수 있는 어노테이션이다.
- 선언할 경우 @Controller, @ControllerAdvice는 사용할 수 있지만, @Service, @Component, @Repository는 사용할 수 없다.
- 여기서는 컨트롤러만을 사용하기 때문에 선언한다.

### @Autowired
- 스프링이 관리하는 빈(Bean)을 주입받는다.

여기서 나오는 Bean이 무엇인지 잘몰라서 아래 게시물에 정리해보았다.

> [Bean이란 무엇일까?](https://learnote-dev.com/java/Spring-Bean이란-무엇일까/)

### private MockMvc mvc
- 웹 API를 테스트 할 때 사용한다.
- 스프링 MVC 테스트의 시작점이다.
- 이 클래스트를 통해서 HTTP GET, POST 등에 대한 API 테스트를 할 수 있다.

### mvc.perform(get("/hello"))
- MockMvc를 통해서 /hello 주소로 HTTP GET 요청을 한다.
- 체이닝이 지원되어 아래와 같이 여러 검증 기능을 이어서 선언할 수 있다.

### .andExpect(status().isOk())
- mvc.perform의 결과를 검증한다.
- HTTP Header의 Status를 검증한다.
- 우리가 알고 있는 200,404,400등의 상태를 검증하는데, isOk()는 200인지 아닌지를 검증한다.

###​ .andExpect(content().string(hello))
- mvc.perform의 결과를 검증한다.
- 응답 본문의 내용을 검증한다.
- Controller에서 "hello"를 리턴하기 떄문에 이 값이 맞는지 검증한다.

이렇게 작성한 테스트를 실행시켜보면, 테스트가 통과 됨을 볼 수 있다. /hello 주소로 GET요청을 하면 hello라는 문자열이 정상적으로 출력되는 것을 직접 눈으로 보지 않고도 우리가 작성한 테스트 코드로 확인 할 수 있는 것 이다.

## 실제로 확인해보기
테스트 코드가 아니라 실제로 hello가 출력되는지 직접 확인하고 싶다면, Application 클래스를 실행 시키면 된다.

```
2022-01-03 21:45:50.916  INFO 66709 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
``` 
그럼 이러한 스프링 부트 로그가 여러개 뜨는데 위 로그를 보면 톰캣 서버가 8080 포트로 실행되었다는 것을 알 수 있다.

이제 웹브라우저를 열어 localhost:8080/hello로 접속해보면, 문자열 hello가 잘 출력되는 것을 확인 할 수 있다.
