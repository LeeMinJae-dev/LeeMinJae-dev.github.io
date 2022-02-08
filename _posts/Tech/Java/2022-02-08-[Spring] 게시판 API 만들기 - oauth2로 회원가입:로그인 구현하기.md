---
title : "[Spring] 게시판 만들기 - oauth2.0으로 회원가입/로그인 구현하기"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
이 게시물은 이동욱님의 "스프링 부트와 AWS로 혼자 구현하는 웹 서비스"를 읽고 공부하며 작성한 게시물입니다.

# oauth 프로필 설정
먼저, 구글 클라우드 플랫폼에서 API 및 서비스에 있는 사용자 인증 정보 탭으로 이동한다.

![image1](/assets/images/tech/Java/api4/image1.PNG)

'사용자 인증 정보' 옆에 씽는 +사용자 인증 정보 만들기에서 OAuth 클라이언트 ID를 눌러 이동한다.
![image1](/assets/images/tech/Java/api4/image2.PNG)

애플리케이션을 "웹 애플리케이션"으로 선택한다.
![image1](/assets/images/tech/Java/api4/image3.PNG)

웹 애플리케이션의 이름을 설정해주고, 승인된 리디렉셕 URI에는 로그인에 성공시 리다이렉션할 URL을 적어준다. 이 주소는 AWS 서버 배포시에는 추가로 주소를 추가해야한다.
스프링 부트 2버전의 시큐리티는 기본적으로 {도메인}/login/oauth2/code/{소셜서비스 코드}로 리다이렉트 URL을 지원하고 있기 때문에, 다음과 같이 입력해준다.
```java
http://localhost/login/oauth2/code/google
```

![image1](/assets/images/tech/Java/api4/image4.PNG)

이렇게 까지 완료하면 이제 사용자인증정보 탭에 클라이언트ID와 클라이언트 비밀번호가 생성된 것을 볼 수 있다.

![image1](/assets/images/tech/Java/api4/image5.PNG)

# application-oauth.properties 파일 생성
먼저 application.properties가 있는 resources 폴더에 application-oauth.properties 파일을 생성해준다. 그리고 해당파일에 아까 받아온
클라이언트ID와 클라이언트 보안 비밀 코드를 다음과 같이 등록 해준다.

```java
spring.security.oauth2.client.registration.google.client-id = {클라이언트 id}
spring.security.oauth2.client.registration.google.client-secret = {클라이언트 비밀번호}
spring.security.oauth2.client.registration.google.scope = profile,email
``` 

스프링 부트에서는 properties 이름을 application-xxx.properties로 만들면 xxx에 해당하는 이름의 profile이 생성되어 이를 관리할 수 있다.

따라서 이를 호출하기 위해서 application.properties에 가서 다음과 같은 코드를 추가한다.

```java
spring.profils.include = oauth

```

주의 할점은 클라이언트 id와 비밀번호가 담긴 이 파일은 깃허브에 그대로 공개하게 되면 개인정보를 가져가는 취약점이 될 수 있기떄문에 .gitignore파일에 등록해서 이 파일이 깃허브로 올라가지 못하도록 해주어야 한다.

.gitignore 파일에 application-oauth.properties를 입력해주면 된다.

이제 본격적으로 User 데이터를 만들고, 구글을 통한 로그인 기능을 구현해보도록 하자.

# 로그인 기능 구현하기
## Domain
먼저 User라는 테이블을 데이터베이스에 만들어주기 위해서 domain 폴더에 user 패키지를 만들고, User클래스를 만들어준다.

### springboot/domain/user/User 
```java
@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    @Builder
    public User(Long id, String name, String email, Role role) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.role = role;
    }

    public User update(String name, String picture) {
        this.name = name;
        this.picture  = picture;
    }

    public String getRoleKey() {
        return this.role.getKey();
    }
}
```
User 테이블을 Entity로 선언해준뒤, 속성으로 id, name, email, picture, role을 갖게 해준다.

role에 쓰인 @Enumerated(EnumType.STRING) 어노테이션은 JPA로 데이터베이스를 저장할때 Enum 값을 어떤 형태로 저장할지를 결정한다.               
기본적으로는 int 숫자가 저장되며, 숫자로 저장되면 우리가 데이터베이스를 볼때 그 값이 어떤 코드를 의미하는지 알 수 가 없기떄문에, String 타입으로 저장될 수 있도록 선언해준다.

Role은 사용자의 권한을 관리할 클래스로, Enum 타입으로 선언해준다.

### springboot/domain/user/Role 
```java
@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER", "일반 사용자");

    private final String key;
    private final String title;
}
```
스프링 시큐리티에서는 권한코드에 항상 ROLE_이 앞에 있어야 인식할 수 있기떄문에, 코드별 키값을 ROLE_GUEST, ROLE_USER와 같이 지정한다.

마지막으로 User를 데이터베이스에 접근할 수 있도록 UserRepository도 선언해준다.

### springboot/domain/user/UserRepository
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}
```
UserRepository에는 마찬가지로 JpaRepository를 상속해주고, 데이터베이스에 이미 있는 이메일인지 체크하기 위한 메서드인 findByEmail을 만들어준다.

이제 User 엔티티 관련한 코드를 모두 작성했으므로, 스프링 시큐리티 설정을 해보자.

## 스프링 시큐리티 설정

먼저 build.gradle 파일에 의존성을 하나 추가해준다.

```java
implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
```

그리고 폴더에 config 폴더를 만들어주고, 그 안에 auth라는 폴더를 또 만들어준다. 시큐리티 관련 클래스는 모두 이곳에서 관리하도록 한다.

그 다음 이곳에 WebSecurityConfigurerAdapter 클래스를 만들어준다.

### springboot/config/auth/WebSecurityConfigurerAdapter
```java
@RequiredArgsConstructor
@EnableWebSecurity
public class WebSecurityConfigurerAdapter extends org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter {
    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable().headers().frameOptions().disable()
                .and()
                .authorizeRequests()
                .antMatchers("/", "/css/**", "images/**", "/js/**", "/h2-console/***").permitAll()
                .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                .anyRequest()
                .authenticated()
                .and()
                .logout().logoutSuccessUrl("/")
                .and()
                .oauth2Login()
                .userInfoEndpoint()
                .userService(customOAuth2UserService);
    }

}

```
* EnableWebSecurity
Spring Security 설정을 활성화 한다.

* .csrf().disable().headers().frameOptions().disable()
h2 콘솔을 사용하기 위해 해당 옵션들을 disable 해준다

* authorizeRequests
URL별 권한 관리를 설정하는 옵션의 시작점으로, authorizeRequests가 선언되어야만 antMatchers 옵션을 사용할 수 있다.

* antMatchers
권한 관리 대상을 지정하는 옵션으로, URL, HTTP 메소드별로 관리가 가능하다. 
여기서 "/"등 지정된 URL은 permitAll()을 통해 전체 열람권한을 주었고, "/api/v1/**"의 주소는 USER권한을 가진사람만 가능하도록 하였다.

* anyRequest 
설정된 값들 이외의 나머지 URL을 나타낸다. 여기서는 authenticated()를 추가하여 나머지 URL들은 모두 인증된 사용자만 가능하게 한다.

* logout().logoutSuccessUrl("/")
로그아웃 기능에 대한 여러 설정의 진입점으로, 로그아웃 성공시 "/"의 주소로 이동한다.

* oauth2Login
OAuth 2 로그인 기능에 대한 설정의 진입점

* userInfoEndPoint
OAuth 2 로그인 성공 이후 사용자 정보를 가져올 때의 설정등을 담당한다.

* userService
소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록한다. 리소스서버에서
사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있다.

여기까지 작성했다면, 아직 CustomOAuth2UserService 클래스가 없어 오류가 날것이다. 아래와 같이 작성해주도록 하자.

###  springboot/config/auth/CustomOAuth2UserService
```java
@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {
    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest
                .getClientRegistration()
                .getRegistrationId();

        String userNameAttributeName = userRequest
                .getClientRegistration()
                .getProviderDetails()
                .getUserInfoEndpoint()
                .getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId, userNameAttributeName,oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);

        httpSession.setAttribute("user", new SessionUser(user));
        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey());


    }

    private User saveOrUpdate(OAuthAttributes attributes) {
        User user = userRepository.findByEmail(attributes.getEmail())
                .map(entity -> entity.update(attributes.getName(), attributes.getPicture()))
                .orElse(attributes.toEntity());
        return userRepository.save(user);
    }
}
```
* registrationId
현재 로그인 진행중인 서비스를 구분하는 코드이다. 지금은 구글만 사용하기 떄문에 없어도 되는 값이지만, 이후 네이버 연동시에 네이버 로그인인지, 구글로그인인지 구분하기 위해 사용한다.

* userNameAttributeName
OAuth2 로그인 진행 시 키가 되는 필드값을 이야기한다. Primary Key와 같은 의미이다. 구글의 경우는 기본적으로 코드를 지원하지만, 네이버 카카오등은 기본 지원하지 않는다.
구글의 기본코드는 "sub"이다.

### springboot/config/auth/dto/OAuthAttributes
```java
@Getter
public class OAuthAttributes {
    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String, Object> attributes) {
        return ofGoogle(userNameAttributeName, attributes);
    }

    public static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity(){
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}

```
* OAuthAttributes
OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스이다. 네이버나 카카오등의 다른 소셜 로그인도 이 클래스를 사용한다.

* of()
OAtuh2User가 반환하는 사용자 정보는 Map의 자료형으로 반환되기 때문에 값하나하나를 변환해주어야한다.

* toEntity()
User 엔티티를 생성한다. OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할때 이며, 가입할때의 기본 권한을 GUEST로 주기 위해서 role값으로 Role.GUEST를 넘겨준다.

### springboot/config/auth/dto/OAuthAttributes
```java
@Getter
public class SessionUser {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```

* SessionUser
세션에 사용자 정보를 저장하기 위한 Dto 클래스이다.

SessionUser에는 인증된 사용자 정보만 필요한데, 그 외에 필요한 정보들은 없으니 name,email,picture만 가져오도록 한다.
굳이 따로 dot을 만들어주는 이유는, 만약 User그대로 세션에 저장하려고 하면, 직렬화를 구현하지 않았다는 에러가 뜨게된다. 그렇다고해서 User에 직렬화 코드를 넣게되면, 만약 엔티티가 @OneToMany와 @ManyToOne으로 자식 엔티티를 갖고있다면, 직렬화 대상에 너무 많은 것들이 포함되어 성능이 느려질 수 있다. 그래서 직렬화 기능을 가진 세션 Dto를 하나 만드는것이 후에 운영이나 유지보수에 유리하다.

## 로그인 버튼 추가하기
작성한 글들의 목록을 보여주는 페이지인 index.mustache에 다음과 같은 코드를 추가한다.

### springboot/resources/templayes/index.mustache
```java
  <div class="row">
        <div class="col-md-6">
            <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#savePostsModal">글 등록</button>
            {{#userName}}
                Logged in as: <span id = "user">{{userName}}</span>
            <a href="/logout" class="btn btn-info active" role="button">Logout</a>
            {{/userName}}
            {{^userName}}
                <a href="/oauth2/authorization/google" class="btn btn-success active" role="button">Google Login</a>
            {{/userName}}
        </div>
    </div>
...
...
<!---목록 출력 영역--->
```

* a href = "/logout"
스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL로, 개발자가 별도로 이 URL에 해당하는 컨트롤러를 만들 필요가 없다. 

* {{#userName}}
해당 값이 존재할 경우에는 #을 사용하여, userName이 존재하면 로그아웃 버튼이 표시된다.

* {{^userName}}
해당값이 존재하지 않을경우는 ^를 사용하여 userName이 존재하지 않으면 로그인 버튼이 노출된다.

* a href="/oauth2/authorization/google"
마찬가지로 스프링 시큐리티에서 제공하는 로그인 URL로, 개발자가 별도의 컨트롤러를 생성할 필요가 없다.

이제 이 mustache 파일이 인식할 수 있도록 IndexController에서 userName을 받아와 Model을 등록해주도록 하자.

### springboot/web/IndexController
```java
@GetMapping("/")
    public String index(Model model) {
        List<PostsListResponseDto> postsList = postsService.findAllDesc();
        model.addAttribute("posts", postsList);

        SessionUser user = (SessionUser) httpSession.getAttribute("user");

        if (user != null) {
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
```

* SessionUser user = (SessionUser) httpSession.getAttribute("user")
앞서 작성된 CustomOAuthUserService에서 로그인 성공 시에 세션에 SeesionUser를 저장하도록 했다. 즉, 로그인이 성공하면 httpSession에서 user값을 가져온다.

이제 홈화면에 접속하면, 로그인이 정상적으로 구현 된것을 확인할 수 있다.

## 웹페이지에서 테스트하기
![image1](/assets/images/tech/Java/api4/image6.PNG)

로그인 버튼이 생성된 것을 볼 수 있는데, 누르면 우리가 자주보던 구글 로그인 화면이 뜨는것을 확인 할 수 있다.

![image1](/assets/images/tech/Java/api4/image7.PNG)

구글 아이디를 선택하면, 정상적으로 로그인이 완료된다.

![image1](/assets/images/tech/Java/api4/image8.PNG)

