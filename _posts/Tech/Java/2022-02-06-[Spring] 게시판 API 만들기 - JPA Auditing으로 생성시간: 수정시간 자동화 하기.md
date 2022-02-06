---
title : "[Spring] 게시판 만들기 - JPA Auditiong으로 생성시간 / 수정시간 자동화 하기"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
이 게시물은 이동욱님의 "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"를 읽고 공부하며 작성한 게시물입니다.

보통 Entity는 해당 데이터의 생성시간과 수정시간을 포함한다. 이러한 생성/수정 정보는 후에 유지보수에 있어서 굉장히 중요한 정보이기 때문에 항상 데이터와 함께 저장해주는게 좋다. 하지만 이를 매번 DB에 삽입할때 날짜 데이터를 등록/수정하는 코드를 만들어 주는 것은 코드도 너무 지저분하고, 너무 많은 노동력이 들어가는 일일 것이다. 그래서 이러한 문제를 해결하고자 JPA Auditing을 사용해서 이를 자동화 해보자.

## LocalDate
Java8부터는 LocalDate와 LocalDateTime이 등장했는데, 그동안 Java의 기본날짜 자료형인 Date가 문제점을 너무 많이 갖고 있어서  Java8 이전에는 JodaTime이라는 오픈소스를 사용해서 이러한 문제점들을 해결했다. 따라서 만약 Java8을 사용하고 있다면, 날짜타입은 무조건적으로 LocalDate와 LocalDateTime을 사용해야 한다.

데이터에 날짜를 추가하기 위해서 BaseTimeEntity 클래스를 만들어주자.

### springboot/domain/posts/BaseTimeEntity
```
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime modifiedDate;
}
```

* @MappedSuperClass
JPA Entity 클래스들이 BaseTimeEntity를 상속할 경우 이 클래스의 변수인 createdDate와 modifiedDate도 칼럼으로 인식하게한다.

* @EntityListners(AuditingEntityListener.class)
BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다.

* @CreatedDate
Entity가 생성되면, createdDate에 저장 시간이 자동으로 저장된다.

* @LastModifiedDate
조회한 Entity의 값을 변경할 때 시간이 자동 저장된다.

이렇게 만들어준 BaseTimeEntity를 Entity인 Posts에 상속해주고, JPA Auditing을 활성화 할 수 있도록 Application 클래스에 활성화 어노테이션을 추가해주면, 자동으로 데이터베이스에 생성시간과 수정시간을 포함하여 저장할 수 있다.

### springboot/domain/posts/Posts
```
@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity{
                    ...
                    ...
}
```
BaseTimeEntity를 상속해준다.

### springboot/Application
```
@EnableJpaAuditing
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```
@EnableJpaAuditing 어노테이션으로 JPA Auditing을 활성화 해준다.

![image1](/assets/images/tech/Java/api3/image1.PNG)

Post 요청을 통해 게시글 데이터를 하나 추가해주면, 데이터베이스에 자동으로 생성시간과 수정시간이 함께 저장된 것을 확인할 수 있다.
