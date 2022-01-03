---
title : "[Java] 웹브라우저로 Hello World 출력해보기"
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
@SpringBootApplication 어노테이션은 이 클래스가 메인 클래스가 된다는것을 알려주는 어노테이션으로, @SpringbootApplication으로 인해 스프링 부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정하게 된다. 
특히나 항상 프로그램은 @SpringBootApplication이 있는 위치부터 설정을 읽어가기 떄문에 이 클래스는 항상 프로젝트 최상단에 위치하여야 한다. 

main 메소드에 있는 SpringApplication.run으로 인해서 내장 WAS (Web Application Server)를 실행한다. 내장 WAS란 별도로 외부에 WAS를 두지 않고 애플리케이션을 실행할 때 내부에서 WAS를 실행하는 것을 이야기 하는데, 이렇게 하면 항상 서버에 톰캣을 설치할 필요가 없게되고, 스프링 부트로 만들어진 jar 파일로 실행하면 된다.

스프링 부트는 내장 WAS를 사용하는데, 어플리케이션을 실행할 때 내부에서 WAS를 실행하기 때문에 외부에 별도의 추가 서버를 두지 않기 떄문에 서버를 배포할때 마다 일일히 WAS의 종류와 버젼, 설정을 일치시키지 않아도 된다.

여기서 WAS라는 개념이 등장하는데 이를 따로 정리해서 따로 게시물로 남겨놓기로 했다.

> [WAS란 무엇일까?](https://www.youtube.com/watch?v=bIeqAlmNRrA&t=2372s)

