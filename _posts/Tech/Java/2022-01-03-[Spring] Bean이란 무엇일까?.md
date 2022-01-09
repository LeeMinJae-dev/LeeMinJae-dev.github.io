---
title : "[Spring] Bean이란 무엇일까?"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
스프링에서 @Autowired를 사용하려다가 Bean이 무엇인지 잘 모르겠어서 Bean에 대해 정리해보려 한다.

## Java Bean
먼저 Java Bean은 특정 형태 클래스를 가르키는 뜻으로, DTO 혹은 VO의 형태가 Java Bean이라고 생각하면 된다.        

필드는 private로 구성되어 getter와 setter를 통해서만 접근할 수 있고, 전달인자가 없는 생성자를 가지는 형태의 클래스이다.

Java Bean의 조건을 정리해보면 다음과 같다. 
- 변수는 getter / setter를 통해서만 접근 가능하도록 private로 선언되어야 한다.
- 생성인자는 전달 인자가 없는 생성자여야한다.

## Spring Bean
Spring에서의 Bean은 스프링 IoC컨테이너가 관리하는 Java 객체를 뜻한다.       
Java Bean과 똑같은 개념이지만, 이 객체는 스프링 IoC컨테이너에서 관리된다. 

스프링 IoC가 관리하는 객체라함은 스프링에 의해 생성되고, 라이프 사이클을 수행하고, 의존성 주입이 일어나는 객체들을 말한다.     
즉, 정리하면 개발자가 관리하는 객체가 아닌 스프링에게 제어권을 넘긴 객체를 스프링에서 Bean이라고 부른다.   


