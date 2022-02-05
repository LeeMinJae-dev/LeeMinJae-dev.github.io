---
title : "[Spring] 게시판 API 만들기 - 데이터 등록"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
이 게시물은 이동욱님의 "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"를 읽고 공부하며 작성한 게시물입니다.

## 시작하며...
이 책으로 바로 시작하려니 사전지식이 너무 부족해서 홍팍님의 유튜브 강의로 대략적인 스프링부트의 기본적인 흐름에 대해서 이해하고 다시 책을 보니, 맨 처음에는 그냥 베끼기만 했던 내용들이 이제 무슨뜻인지 이해가 되기 시작했다. 다시 처음부터 게시판 웹서비스를 만들어보도록 하자.

이번에는 View가 없이 http요청을 통해 Json형태로 정보를 받아오는 Rest api를 만들도록 한다.
게시판 웹페이지를 만든다고 생각하고 CRUD 네 기능을 모두 구현해보자.

# 새글 등록하기
## Domain
도메인이란 소프트웨어에 대한 요구사항 및 문제영역이라고 생각하면 되는데, 여기에 있는 클래스들은 대부분 Entity가 된다. Entity는 데이터베이스에 저장된 값으로, 이 값은 함부로 접근하거나 변경되어서는 안되므로, 항상 이 값을 꺼내올때는 Entity 자체에 접근 하는게 아니라, Dto라는 다른 형태로 가져와 처리를 하고 다시 Entity화 하여 저장하도록 한다.

그럼 먼저 게시판의 게시글이 될 Posts 클래스부터 구현해보자.

### springboot/domain/posts/Posts
```
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;
    
}
```
@Entity 어노테이션은 이 클래스가 Entity가 될 클래스임을 명시하는 어노테이션으로, 이 어노테이션을 사용하면 테이블과 해당 클래스가 링크 된다. 따라서 Posts라는 테이블이 생성되고, @Column 어노테이션을 클래스 변수들에게 지정해 줌으로써 해당 변수들은 모두 칼럼이 된다. 

id에 붙은 @Id는 이 값이 이 테이블의 PK, Primary Key가 된다고 선언하는 것인데, 추가로 GeneratedValue 어노테이션을 사용해주면, 이 PK를 스프링 부트가 알아서 생성한다. 직접 PK를 개발자가 지정해주면 오류가 발생할 확률이 매우 크므로, 특정상황을 제외하고서는 GenerationType.IDENTITY를 사용하여 자동생성 해주도록 해야한다.

이 Posts 클래스는 게시물에 대한 데이터가 되어야 하므로 PK인 id와 게시물의 제목인 title, 그리고 내용인 content, 작성자인 author를 칼럼으로 갖는다. @Column은 특정 세부설정을 할때는 선언해주어 title과 content와 같이 설정해주어야 하지만, author 처럼 따로 설정해야할 세부 설정이 없다면 @Column 없이도 알아서 해당 변수를 칼럼으로 인식한다. 

이 변수명들은 그대로 칼럼의 이름이 되며, 카멜케이스 방식으로 사용하면 알아서 언더바 형식으로 변환된다.
ex) mainTitle => MAIN_TITLE

Entity를 만들었으니, 이 Entity를 데이터베이스에 접근하게 해줄 JpaRepository를 생성해주자.


## Repository
Repository는 직접적으로 데이터베이스에 접근할 수 있도록 해주는 역할을 하는데, 굳이 데이터베이스에 복잡하게 SQL명령을 보내지 않아도 Jpa를 사용하면, 객체지향적으로 훨씬 편하게 데이터베이스를 관리할 수 있다.

### springbood/domain/posts/PostsRepository
```
public interface PostsRepository extends JpaRepository<Posts, Long> {

}
```
인터페이스로 PostsRepository를 생성해 준 뒤, JpaRepository를 <Entity 클래스, PK타입>과 함께 상속해주면 Posts 클래스가 데이터베이스에 접근할 수 있도록 하는 Repository가 된다.

이제 해당 클래스를 통해서 save나 delete등의 메소드를 사용해 해당 엔티티의 데이터베이스에 접근할 수 있게 되었다.

이제 본격적으로 데이터를 등록, 삭제, 변경하는 Rest API를 만들어보자.

## Service
서비스 메소드는 도메인에 접근하는 기능들의 순서를 보장해주며, 이를 @Transactional 어노테이션을 사용함으로써, 데이터베이스에 접근하는 도중 오류가 발생하면 해당 기능을 모두 완수하지 않은 상태로 멈추는게 아니라, 아예 데이터베이스에 접근하기 전의 상태로 돌아가게 해주는 역할을 해서 데이터자체에 오류가 섞이지 않도록 해준다.

### springboot/service/posts
```
@RequiredArgsConstructor
@Service
public class PostsService {

    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto) {
        return postsRepository.save(requestDto.toEntity()).getId();
    }
}
```
마찬가지로 @Service를 통해서 이 클래스가 서비스 메서드임을 선언해주고, 아까 생성해준 Repository를 선언해준다. final을 사용하고 @RequiredArgsConstructor를 사용해주면, 롬복이 final 변수에 대한 생성자를 만들어주기 때문에 따로 @AutoWired를 사용하지 않고도 Bean을 주입할 수 있다.

먼저 테이블에 데이터를 등록하는 기능을 위해서 save 메서드를 만들어준다. 여기서 save 메소드는 매개변수로 RequestDto의 자료형을 받는데, 이 RequestDto는 앞서 말했듯이 데이터에 직접 접근하지 않기 위해서 데이터와 비슷한 모양의 데이터를 만들어준다고 생각하면 편하다. 

만약 우리가 어떤 밑그림만 그려져 있는 그림을 칠한다고 생각해보자. 바로 밑그림에다가 색부터 칠한다면, 색을 칠하고보니 잘 안어울리거나 잘못된 색을 칠했을때는 이미 원본이 훼손되었기 때문에 돌이킬 수 없을 것이고, 돌이킬 수 있더라도 많은 고생과 시간이 소요될 것 이다. 하지만 그 밑그림을 복사해서 한번 칠해보고 오류가 없는 것이 확인되면, 복사본을 토대로 원본에 색을 안전하게 입힐 수 있을 것이다. 여기서 원본이 Entity라면 Dto가 바로 이 원본의 복사본이다.

### springboot/web/dto/PostsSaveRequestDto
```
Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Posts toEntity() {
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```
Dto 클래스의 모습을 보면 Entity 와 거의 똑같은 모습을 볼 수 있다. 앞서 말했듯이 데이터의 손상이나 오류를 방지하기 위해 Entity와 아주 비슷한 모습의 형태로 Entity를 가져와 가공하고 다시 toEntity() 메서드를 통해 다시 Entity로 변환하여 데이터베이스에 접근하게 되는 것이다. 

여기서 PostsRequestDto를 보면, toEntity 메서드를 제외하고는 Posts와 아예 똑같은 것 같은데 굳이 '비슷한'이라는 단어를 사용한 이유는, 만약 Dto에 옮겨야할 데이터를 굳이 모두 가져올 필요가 없다면 title만 선언해준다던가 하는 식으로 Entity와 모습이 살짝 다른경우도 있기 때문이다.

이렇게 PostsRequestDto를 통해서 Service는 Dto 형태의 Post 요청을 받아 Repository로 넘겨 이를 데이터베이스에 저장하도록 한다. 그럼 JSon형태의 요청을 받아오기 위한 마지막 단계를 알아보자.

## Controller
Contoller는 우리가 요청을 보내고 받는 부분으로, 웹페이지를 보는 View에서 서버에게 보내는 모든 요청은 Controller를 통해 서버에 전달 된다. 따라서 사용자가 보는 View에서 가장 가까이있는 클래스이다.

### springboot/web/PostsApiController
```
@RequiredArgsConstructor
@RestController
public class PostsApiController {

    private final PostsService postsService;

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto) {
        return postsService.save(requestDto);
    }
}
```
@RestController 어노테이션을 선언함으로써, 이 클래스가 RestAPI를 만들기 위한 Contoller부분임을 선언한다. 이로써 web에서 우리가 요청을 보내면 Controller가 가장 먼저 받아 데이터베이스까지 요청을 전달해준다.

마찬가지로 PostsService 객체를 만들어주고 여기에 빈을 주입하기 위해 @RequiredArgsConstructor를 선언해준다.

우리는 새로운 글을 등록해야 하므로 Post 요청을 보내야 한다. 따라서 글을 등록하는 save 메서드는 @PostMapping 어노테이션을 사용해 Post 요청을 어노테이션 안에 있는 주소로 받는다. 따라서 우리는 /api/v1/posts로 Post 요청을 양식에 맞게 Json 데이터로 만들어 보내면, 해당 요청을 받아 보낸 요청에 담긴 데이터를 데이터베이스에 추가할 것 이다.

@RequestBody 어노테이션이 바로 받아온 Json 데이터를 스프링부트의 변수에 담을 수 있도록 해주는 어노테이션으로, 우리가 Json 요청을 아래와 같이 보내면,

```
{
    "title": "제목",
    "content": "내용",
    "author": "이민재"
} 
```

알아서 PostsSaveRequestDto 변수에 해당 값들을 할당한다. 이렇게 받아온 dto를 Service로 보내고, Service가 dto를 Entity형태로 바꾸어 Repository로 넘겨주면 Repository가 해당 데이터를 데이터베이스에 저장한다.


지금까지 배운 모든 구조를 간략화해서 데이터베이스에서 우리가 요청을 받고 요청을 보내는 View까지의 단계를 대략적으로 나타내자면, 다음과 같다.


> Json 데이터를 담은 요청 -> (dto로 변환) -> Controller -> Service -> (Entity로 변환) -> Repository -> 데이터 베이스 

만약 반대로 데이터베이스로 부터 데이터를 얻어온다면,

> 데이터베이스 -> Repository -> (Entity) -> Service -> (dto로 변환) -> Controller -> 요청에 따른 데이터

가 될 것이다.

이로써 데이터 등록을 완료했다. 크롬 Talend를 통해 Post요청을 해당 주소로 보내면, 

![image1](/assets/images/tech/Java/api1/image1.PNG)

![image2](/assets/images/tech/Java/api1/image2.PNG)

위와같이 해당 요청이 정상적으로 200의 Response를 보내며 등록되는 것을 볼 수 있다.

h2-console을 사용해서 Posts의 데이터베이스를 조회해도 다음과 같이,
![image3](/assets/images/tech/Java/api1/image3.PNG)

데이터가 정상적으로 등록된 것을 볼 수 있다.
