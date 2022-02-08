---
title : "[Spring] OAuth2 로그인 후에 js/css 파일이 노출되는 에러"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
# 에러
![image1](/assets/images/tech/Java/apiError1/image1.PNG)

위 사진과 같이 로그인후에 홈으로 가지지 않고 js파일이 노출되는 현상이 발생했다.

# 원래 코드
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
                    .antMatchers("/", "/js/app", "/images/**", "/js/**", "/h2-console/***")
                    .permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                    .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                    .userInfoEndpoint()
                    .userService(customOAuth2UserService);
    }
}
```

# 해결 과정
구글링 결과 WebSecurityConfigurerAdapter의 configure에 다음과 같은 메서드를 추가하면 된다고 해서 따라해봤는데,
```java
@Override public void configure(WebSecurity web) throws Exception {
        web.ignoring()
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }
```
위와 같이 해도 해결이 되지않고 동일한 에러가 발생했다.


그래서 작성한 코드중에 에러를 일으킬만한 부분이 어디인지 찾아보니, configure 부분의 antMatcher() 메서드가 수상해보였다.

antMatcher()는 권한 관리 대상을 지정하는 옵션인데, 디렉토리 권한 허용설정이 제대로 안되고 있는것이 의심이 됐다. 위의 해결방법도, 결국 허용해주지 않은 js나 css폴더에서 발생하지 않는 에러를 해결하기 위해서 web.ignoring()을 사용한 것 같았다. 그래서 노출되는 js 파일의 경로를 antMatcher에 추가해보기로했다.

> 노출된 주소 :​ http://localhost:8080/static/js/app/index.js 

# 변경된 코드
```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                ...
                ...

                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/js/app", ,"/static/**", /images/**", "/js/**", "/h2-console/***")
                    .permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
               ...
               ...
    }
```
antMatchers 부분에 /static의 경로도 추가해 권한을 허용해주었다.

그리고 다시 서버를 재시작하니, 로그인 후에 정상적으로 내가 설정해둔 홈으로 다시 리다이렉션 되는 것을 확인 할 수 있었다.
