---
title: "MapStruct 사용해보기!"
categories: java
toc: true
toc_label: "On this page"
toc_sticky: true
---
## 서론
기존 코드에서 엔티티 안에 toDto라는 메서드를 사용하여 Entity를 Dto로 변환하여 사용하곤 했는데, 의존성을 줄이는 리팩토링을 하다보니 이 부분이 문제라는 생각이 들었다.

이렇게 하면 결국 Entity와 Dto가 의존관계를 갖게 되는 것이기 때문에 Dto의 코드가 변경되거나 해당 엔티티가 다른 Dto를 반환하고자 할때 엔티티 내부의 코드를 수정해야 할 가능성이 생긴다.

따라서 이를 해결하고자 MapStruct 라이브러리를 적용해보기로 했다.

## 근데, Mapper 쓰는게 좋은거 맞아?
Mapper를 적용하기 위해서 찾다보니, 사실 Mapper를 적용하는가 마는가자체가 호불호가 있는 상황이었다.

초기에는 괜찮지만, 그 관계가 복잡해질 수록 오히려 Mapper를 적용 하는것이 더 많은 비용을 사용하기 때문에 반대한다는 이유도 있었고, 코드량이 줄어들고 가독성이 확연히 좋아지기 때문에 Mapper를 적용하는게 좋다는 이유도 있었다.

하지만 어쨌든 나는 아직 사용을 해보지도 않은 상태에서 단점들을 읽고 이를 결정하기 보다는, 우선 적용해보며 장점과 단점을 직접 느껴보기로 했다.

## DTO란?
DTO는 **Data Transfer Object**로, 데이터 교환을 위해 사용하는 객체다.

도메인 객체에 직접 접근하는 것은 데이터가 손상될수 있기 때문에, DTO를 사용하여 도메인 객체를 한번 변환하여 주는 것이 안전하다.

뿐만아니라, DTO를 사용하여 View에 보여줄 데이터를 한번 걸러줄 수 있는데, 이를 통해 숨겨야할 데이터를 숨기거나, 불필요한 데이터를 가져오지 않음으로써 성능상의 이점을 볼수도 있다.

예를들어 다음과 같은 유저 정보가 있다고 할때, 

```
public class User {

    public Long id;
    public String email;
    public String password; 
}
```
만약 User 자체를 return하게 되면 사용자들에게 보여선 안될 password값도 같이 넘어가게 된다.

 > 물론 암호의 경우는 따로 암호화등이 필요하겠지만 이해를 돕기 위한 예시를 위해 위와 같은 예시를 들어보았다.
 
 하지만 DTO를 사용하면 어떻게 될까?
 
 ```
 public class UserDto {

    public final long id;
    public final String email;

    public static UserDto from(User user) {
        return new UserDto(user.getId(), user.getEmail());
    }
}
 ```

위와같은 Dto를 통해서 User 객체를 UserDto로 변환함으로써 password와 같은 숨겨야할 정보를 제거하고 사용자에게 보여주기 때문에 불필요한 정보를 가려 데이터를 안전하게 지킬 수 있다.

## 현재 코드의 문제점

위의 예시를 예로 들면 현재 코드는 다음과 같이 구성되어있다.

```
public class User {

    public Long id;
    public String email;
    public String password; 
    
    public UserDto toDto() {
    	return new UserDto(this.id, this.email);
    }
}
```
toDto() 메서드가 엔티티 안에 존재해서, 만약 UserDto와 다른 Dto로 변환하게 하고싶다면 엔티티 코드를 변경해야한다. 이것은 코드 유지보수에 있어 매우 좋지 않은 방법이다.

뿐만아니라 불필요한 반복적인 코드작성을 통해 생산성이 떨어지는 건 물론, 재사용성도 몹시 떨어진다.

따라서 Mapper 라이브러리의 도움을 받아, 조금 더 객체 지향적인 방법으로 이를 변경해보려 한다!


## MapStruct vs ModelMapper
Mapper를 위한 라이브러리는 대표적으로 위 두개의 라이브러리가 있다.

ModelMapper의 경우 내부적으로 Reflection을 사용하기 때문에 성능이슈가 있으며, 컴파일 시점에서 오류를 잡을 수 없기 때문에 위험하다고 한다. 

사실 내 어플리케이션 크기가 ModelMapper의 성능상 이슈까지 고민해야할 부분은 아닐 것 같지만, 오류중에 가장 위험한 오류가 컴파일 시점에서잡아낼 수 없는 오류라고 생각하기 때문에 위 두 문제점에서 비교적 안전한 MapStruct를 적용해서 맵핑을 해보기로 했다.


## MapStruct 사용하기
MapStruct를 사용하기 위해서, Gradle에 의존성을 추가해주어야 한다. 물론 Maven을 사용하는 경우는 Maven에 의존성을 추가해주어야 한다!

```
implementation 'org.mapstruct:mapstruct:1.4.2.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
```

### *Lombok을 같이 사용할때 주의점*
Lombok을 같이 사용하는 경우, 

```
annotationProcessor 'org.projectlombok:lombok-mapstruct-binding:0.2.0'
```

의 의존성을 추가해주면 Lombok과 MapStruct를  함께 사용할때 발생하는 순서 충돌 문제를 해결해준다고 한다.

위 의존성을 추가해주지 않는다면, 반드시 Lombok 의존성 뒤에 MapStruct 의존성을 선언 해주어야한다. 기본적으로  MapStruct가 Lombok의 getter, setter, builder를 이용하여 생성되므로, MapStruct가 Lombok보다 먼저 선언된다면 정상적으로 실행 되지 않는다.


그럼 위에서 언급한 예제를 MapStruct를 통해 변환해보도록 하자.

### UserMapper
```
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDto toDto(User user);
}
```
먼저 위와 같이 Mapper 인터페이스를 선언해준다. 중요한건 componentModel 파라미터를 spring으로 설정해주어야 UserMapper가 빈으로 등록되어 의존성을 주입받을 수 있다.

이제 우리는 Mapper 객체를 사용하여 엔티티를 Dto로 변환할수 있게 되었다!

이제 빌드를 한번 돌려보면, 다음과 같은 코드가 generated 폴더에 생서된다.

### UserMapperImpl
```
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2023-02-20T20:06:15+0900",
    comments = "version: 1.4.2.Final, compiler: IncrementalProcessingEnvironment from gradle-language-java-7.4.1.jar, environment: Java 11.0.16.1 (Azul Systems, Inc.)"
)
@Component
public class UserMapperImpl implements UserMapper {

    @Override
    public UserDto toDto(User entity) {
        if ( entity == null ) {
            return null;
        }

        UserDtoBuilder userDto = UserResponseDto.builder();

        UserDto.entity( entity );

        return userDto.build();
    }
}
```
이렇게 우리가 아까 만들었던 UserMapper 인터페이스를 구현한 구현체인 UserMapperImpl이 생성되고, 이렇게 생성된 코드를 사용하는 방식이기 때문에 내부적으로 Reflection을 사용하는 ModelMapper보다 성능상 우위를 가질 수 있는 것이다.

그럼 이제 아래와 같은 코드로 Mapper를 사용한 변환을 사용할 수 있게 된다.

```
public class UserService {
	private final UserRepository userRepository;
	private final UserMapper userMapper;
	
	@Transactional
	public UserDto getUser(Long id) {
		User entity = userRepository,findById(id);
		return userMapper.toDto(entity);
	}
}
```
따라서 toDto와 같이 도메인 객체를 DTO 객체로 변환하는 책임을 Mapper에 줌으로써 조금 더 객체지향스러운 코드가 완성 되었다.

## 결론
사용해보니 좀 더 책임이 분리되고 객체 지향적인 느낌이 드는 것 같다.

아직 복잡한 맵핑관계를 경험해 보지 않아서 모르겠지만, ModelMappper의 단점을 상당부분 개선한 MapStruct를 사용한다고 가정하면 Mapper 라이브러리를 사용하는 것은 코드량도 줄일 수 있고, 훨씬 객체 지향스러운 방법인 것 같다!

당분간은 아마 Mapper를 사용해서 개발할지도...?


