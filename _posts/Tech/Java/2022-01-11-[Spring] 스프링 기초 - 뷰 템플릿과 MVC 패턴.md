---
title : "[Spring] 스프링 기초 - 뷰 템플릿과 MVC 패턴"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
이 게시물은 홍팍님의 "스프링 부트입문"을 시청하며 공부한 내용을 작성한 글입니다.

>> **출처**
>> [홍팍 유튜브 링크](https://www.youtube.com/watch?v=Y_gkH0nLMY8&list=PLyebPLlVYXCiYdYaWRKgCqvnCFrLEANXt&index=4)

## 시작하며...
게시판을 만들며 스프링을 배워보려 했는데, 그러려고 산 책이 조금 스프링에 대한 기본지식을 아는 사람을 대상으로 쓴 책 같아서 설명이 부족한것 같아 스프링의 기본적인 기능들을 숙지해보려고 강의를 찾아보던 중 홍팍님의 스프링 부트 입문 강의가 좋은 것 같아 영상을 보며 공부해보려 한다.

## 뷰 템플릿과 MVC 패턴
우리가 만약 어떤 웹페이지에 로그인을 한 상태라면, 웹페이지에 표시되는 내 정보, 예를들어 내 이름이나 아이디 등 이러한 정보들은 나만을 위한 정보일 것이다. 그렇다면 이 정보들이 담긴 화면을 모두 하나 하나 웹페이지로 저장한 것일까? 그렇게 한다면 가입자가 생길때마다 가입자에 맞는 정보가 담긴 웹페이지를 하나하나 코딩해주어야 할 것 이다.

그래서 웹페이지를 만들 때는 뷰템플릿이란 기술을 사용한다. 뷰템플릿이란 화면을 담당하는 기술로, 틀이되는 화면을 만들고 안에 있는 몇몇 정보만을 변수로 만들어 뷰템플릿하나로 정보가 다른 여러 화면을 만든 것과 같은 효과를 낼 수 있다. 

이러한 뷰템플릿에서는 컨트롤러와 모델이 있는데, 컨트롤러는 로직을 담당하고 모델을 데이터를 담당하게 된다. 그렇게 되면 뷰, 모델, 컨트롤러 세가지가 각각 임무를 맡아 하나의 프로그램을 동작하게 되는데 이를 MVC 패턴이라고 한다.

## Hello World 출력해보기
먼저 resource 폴더에 있는 templates 폴더에 새로운 파일을 하나 만들어준다. greetings.mustache

![image1](/assets/images/tech/Java/hongpark1/image1.PNG)

그럼 이렇게 파일을 인식할 수 없다는 창이 뜨는데, mustache 플러그인을 설치해주어야 정상적으로 이 파일을 인식할 수 있다.

커맨드 + 시프트 + A를 눌러 인텔리제이의 액션 검색창을 열어주고, plugins를 입력한 후에 마켓플레이스에서 mustache를 인스톨 해주면 올바르게 파일이 mustache 파일로 인식된다.

![image1](/assets/images/tech/Java/hongpark1/image2.PNG)

이제 만들어준 greetings.mustache 파일에 아래와 같은 html을 입력해준다.
```
<html>
    <head>
        <meta charset="UTF-8">
        <meta name = "viewport"
            content="width = device-width, user-scalable = no, initial-scale = 1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <h1>Hello World</h1>
    </body>
    </head>
</html>
```

이렇게 했으면, 이 html코드를 브라우저로 전송할 controller를 만들어보자.

main 폴더에 controller 폴더를 만들어주고, GreetingsController라는 클래스를 하나 만들어준다.

```
import org.springframework.stereotype.Controller;

@Controller
public class GreetingsController {

    public String greetings() {
        return "greetings";
    }
}
```
위와 같이 코드를 입력해주면, @Controller를 통해 스프링부트가 이 클래스를 컨트롤러로 인식하고, greetins() 함수가 파일명이 "greetings"인 파일을 찾아 브라우저로 전송한다.

그럼 이렇게 만든 컨트롤러가 제대로 작동하는지 main파일에 있는 Application 파일을 실행 시킨뒤, http://localhost:8080/greetings.mustache 주소로 들어가 올바르게 브라우저로 전송이 되었는지 확인해보자.

![image1](/assets/images/tech/Java/hongpark1/image3.PNG)

그럼 위처럼 NotFound 404 에러가 뜬다. 아직 파일을 찾을 수 없다는 것으로, 브라우저에 mustache 파일을 전송하려면 추가적인 코드가 더 필요하다.

```
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class GreetingsController {
    
    @GetMapping("/greetings")
    public String greetings() {
        return "greetings";
    }
}
```
위 코드를 보면 @GetMapping 어노테이션을 사용해서 해당 함수가 반환할 값을 표시할 주소를 명시해준다. 그럼 우리는 해당 주소로 이동하면 해당 함수가 리턴하는 greetings.mustache를 확인 할 수 있게 되는 것이다.

이제 다시 Application 파일을 실행시키고 http://localhost:8080/greetings 로 이동하면 아래 처럼 올바르게 우리가 적은 html코드가 출력되는 것을 확인 할 수 있다. 
![image1](/assets/images/tech/Java/hongpark1/image4.PNG)


