---
title : "[Java] assertj 설정 방법"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
## 문제
 테스트코드를 작성해보려고 따라하는데, assertThat()이 안되서 한참을 찾았다.
 
 처음에는 assertThat이 Junit 자체 메소드인줄 알고 Junit 버젼이 달라서 그런가했는데, assertJ라는 다른 라이브러리였다. Junit과 함께 편리하게 사용하게 해주는 라이브러리로, 테스트코드 작성을 훨씬 편리하게 도와준다.
 
 이러한 assertJ를 사용하려면 build.gradle에서 라이브러리 의존성에 추가해주어야한다,
 
 ## 해결 방법
 build.gradle 파일에 있는 'dependency' 부분에 아래와 같은 코드를 추가해준다.
 
 ```
 testImplementation "org.assertj:assertj-core:3.20.2"
 ```

## 알게된 점
다른 언어들과 다르게 자바는 라이브러리의 의존성도 관리를 해주어야 한다는 걸 알았다.

효율적인 프로그램을 위해 의존성에 추가해준 코드에 따라 어떤 시점에 이 라이브러리를 호출 할지를 명시해주어야
그 상황에 라이브러리를 실행시키고 이 라이브러리를 결과물에 포함할지 안할지 정해서 프로그램이 동작한다.

의존성에 추가할 수 있는 코드는 아래의 4개이다.

 1. Implementation
내부적으로만 사용되고 사용자에게는 의존성을 노출시키지 않게 선언한다.
의존 라이브러리를 수정해도 본 모듈까지만 재빌드한다.

 2. compileOnly 
컴파일 타임에 필요한 라이브러리
컴파일 시에만 빌드하고 빌드 결과물에는 포함하지 않는다.

 3. runtimeOnly
런타임 시점에만 필요한 라이브러리

 4.testImplementation
테스트할 때에만 사용된다.

 
 
