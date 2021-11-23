---
title : "[Java] 플레이그라운드 with TDD, 클린코드 - AssertJ (다시 1)"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
## 다시...
 원래는 자바를 공부하면서 나쁜 습관들이 들까봐 애초에 클린 코드로 버릇을 들으려고 강의를 찾다가     
플레이 그라운드에 있는 이 강의가 유용하다고 해서 봐서 결제해서 듣고 있었는데, 앱개발 한다고 결제해놓은.       
강의도 내팽겨 치고 열심히 개발을 하고 왔다. 근데 우테코 붙고 자바 공부도 할겸, 프리코스 미리 예습하려고 찾아보는데
이 강의를 프리코스에서 사용하고 있었고, 이 강의 강사분이 바로 우테코 캡틴분이었다... 헐;;

그래서 뭐 나름 결제도 해놨겠다, 프리코스도 이 내용이겠다. 다시 신나게 자바 좀 공부해보려 한다.       
다시 할라니까 기억이 가물가물해서 다시 처음부터!!

## 단위테스트
 초반 부분은 좀 기억이 나서 한번 훑고만가려 한다.      
 
 자바는 절차지향 언어이기 때문에, 대부분의 기능을 클래스로 구현해놓고 그 클래스들을 메인 메서드에서 구현하는 방식으로 동작한다.
메인 메서드는 프로그램을 시작할 뿐만 아니라 구현한 프로그램을 테스트할때 중간중간 돌려보면서 사용되는데, 그냥 간단하게 내가 맨날
자바를 사용할 때 중간중간 지금까지 한게 맞나 테스트겸 빌드 해보는거랑 같다.

하지만 이러한 메인 함수에서 하는 테스트는 몇가지 문제가 있다.

 1. 프로덕션 코드와 테스트 코드가 하나에 존재한다. 
이건 앱개발 할때도 느꼈던건데, 개발 초반 부분에는 그냥 메인함수를 돌려보면 되지만, 나중에 규모가 커지면 그 기능 하나를 테스트 하기
위해서 전체를 돌려야 하니까 번거롭게 로그인도 해야하고 그 기능이 있는 부분까지 도달하며 테스트해야해서 시간도 많이 들 뿐만 아니라, 
그 기능만이 확실히 작동하는지 독립적으로 살펴볼 수가 없다.

 2. 테스트 코드가 실제 서비스에 같이 배포된다.
이것도 마찬가지로 앱개발할때 테스트로 넣어둔 기능을 깜빡하고 실제 사용자들 버젼에 배포한적이 여러번 있다.

 3. main method 하나에서 여러 개의 기능을 테스트 하기 때문에 복잡도가 증가한다.
1번에서 느꼈던 내용에 포함된 내용인것 같다. 여러개가 테스트 되다 보니까 헷갈린다.

 4. method 이름을 통해서 어떤 부분을 테스트 하는 것인지 알기가 힘들다.
main 함수로 테스트를 하면 협업간에 이게 테스트 코드인지 아닌지 모르니까 헷갈릴 것 같다.

 5. 테스트 결과를 사람이 수동으로 확인해야한다.
내가 어떤 기능을 테스트하려고 불필요한 로그인을 하고 버튼을 누르고 했던 내용을 말한다.

이렇게 main method를 사용한 테스트를 해결하기 위해 등장한게 바로 JUnit이다.

## JUnit
 JUnit은 단위 테스트 프레임워크로, System.out으로 번거롭게 테스트 케이스를 확인하지 않도록 도와주는 도구이다.
프로그램 테스트 시 걸리는 시간도 관리 할 수 있도록 해주며, 오픈소스 형태로 대부분의 IDE에 포함되어있다.
개발을 진행하면서 어느정도의 개발이 진행되면 반드시 프로그램에 대한 단위테스트를 진행해주어야 한다.

## AssertJ
AssertJ는 Junit과 마찬가지로 테스트를 위한 라이브러리로, Junit과 함께 사용하면 테스트 코드를
훨씬 가독성있고 효율적으로 작성할 수 있도록 도와준다. 

### 예제

```
class aboutStringTest {
    @Test
    void replace(){
        String actual = "abc".replace("b", "d");
        assertThat(actual).isEqualTo("adc");
    }
}
```
 위 코드는 assertJ에서 제공하는 assertThat() 메소드를 사용해서 테스트 해본 예제이다.      
actual 메소드가 isEqualTo("adc"), 즉 adc 와 같은 값이 출력되는지를 테스트하게 된다.

만약 assertJ가 아닌 Junit 으로 위와 동일한 테스트를 진행한다면,

```
class aboutStringTest2 {
    @Test
    void replace(){
        String actual = "abc".replace("b", "d");
        assertEquals("adc", actual);
    }
}
```
위와 같은 코드가 될 것이다. Junit의 경우, assertEquals() 메소드를 사용해서 비교하게 되는데, 첫번쨰 파라미터에는 기댓값인 adc가 들어오고 두 번째 파라미터에는 우리가 테스트 할 값인 actual이 들어온다.     
이렇게 봐서는 딱히 assertJ를 사용해야할 이점이 눈에 띄게 보이지 않는다. 하지만 테스트 조건을 늘리게 된다면,

1. assertJ의 경우

```
class aboutStringTest {
    @Test
    void replace(){
        String actual = "abc".replace("b", "d");
        assertThat(actual).isEqualTo("adc")
                .startsWith("a")
                .contains("d");
    }
}

```

2. Junit의 경우

```
class aboutStringTest2 {
    @Test
    void replace(){
        String actual = "abc".replace("b", "d");
        assertEquals("adc", actual);
        assertTrue(actual.startsWith("a"));
        assertTrue(actual.contains("d"));
    }
}
```

두 코드를 비교해보면, assertJ가 좀 더 많은 조건을 추가하기 쉽고 구어체에 가까운 느낌이라
헷갈리지 않고 메서드 체이닝을 하기 유리하다.

그럼 assrtJ를 사용하기 위해서 기본적인 몇가지 메소드들을 알아보자.

### Test Fail Message

JUnit5 에서는 마지막 인자값에 as()를 통해 테스트 실패 메시지를 명시해주어 테스트가 실패했을때 출력되는 메시지를 만들 수 있다.

```
class aboutStringTest {
    @Test
    void split(){
        String[] actual = "1,2".split(",");

        assertThat(actual).as("acutal의 값").contains("1");

    }

}

>>> java.lang.AssertionError: [acutal의 값] 
Expecting:
 <["1", "2"]>
to contain:
 <["3"]>
but could not find:
 <["3"]>

```
이렇게 as() 메서드를 사용해주면, 이 테스트에 대한 설명을 앞에 출력해준다. as()메서드는 무조건
assertion 전에 사용해주어야한다.

### Test Filtering Assertions
 특정 filter를 자바 람다식을 이용하여 표현할 수 있는 유용한 기능이다.
 
```

class aboutStringTest {
    @Test
    void split(){
        String[] actual = "lee,kim,park,".split(",");

        assertThat(actual).filteredOn(x -> x.contains("k")).containsOnly("kim","park");

    }

}
```

위 코드를 보면 filteredOn안에 람다식을 넣어서 k가 포함된 값만을 추출해준뒤, containsOnly를 통해 올바르게 k가 포함된 값인 kim과 park이 결과값으로 나왔는지 확인해준다.
이처럼 filteredOn을 사용하면 특정 케이스들만을 골라서 테스트 해볼 수 있다.

### Exception 처리 테스트
예외처리에 대해서는 먼저 throws에 대해 알아야 한다.

보통 자바에서는 try ~ catch 문을 사용해서 try에서 에러를 감지해서 cahtch 안에서 해당 오류를 처리한다.       
이렇게 예외를 처리하는 방법에는 한가지 방법이 더 있는데, 바로 throws를 사용하는 것이다.       
try ~ catch는 에러가 발생했을떄 바로바로 자신을 예외처리했다면, throws는 메소드 단위로 이 메소드를 사용하는 곳으로 책임을 전가한다.     
한번 다음 예제를 보자.

```
import java.io.IOException;

public class Example{
    public static void main(String[] args){
        
        byte[] Recive = new byte[128];
        
        Read(Recive);
    }
    
    public static void Read(byte[] buffer) throws IOException
    {
        System.in.read(buffer);
    }
}
```

이렇게 작성하고 코드를 실행시키면, Read 메서드를 정의하는 부분에서 에러가 발생하지 않고, Read를 사용한 main 메서드에서 에러가 발생한다.      
메서드를 사용하는 곳에서 예외를 처리하라고 throws를 사용해서 표기해주었기 떄문이다.

따라서 아래의 코드처럼,

```
import java.io.IOException;

public class Example{
    public static void main(String[] args){

        byte[] Recive = new byte[128];
        try{
            Read(Recive);
        } catch (IOException e){

        }

    }

    public static void Read(byte[] buffer) throws IOException
    {
        System.in.read(buffer);
    }
}
```
이렇게 main 메서드에서 try ~ catch를 사용해줌으로써 예외를 처리해줄 수 있다.

또 throw라는 예약어를 사용해서 강제로 오류를 발생시킬 수 도 있는데, 아래의 코드처럼 사용하면 된다.
```
public class Example{
    public static void main(String[] args){
       
        throw new Exception();
        
    }

}
```
이렇게 하면 throw new Exception 위치에서는 Exception()이라는 오류가 발생한다.       
마찬가지로 try ~ catch를 사용해 예외처리 할 수 있다.
```
import java.io.IOException;

public class Example{
    public static void main(String[] args){
        try{
            throw new Exception();
        }catch (Exception e){

        }
    }

}
```

그럼 이제 테스트코드에서의 예외처리에 대해서 알아보자.

assertJ는 가독성있는 코드를 위해 assertThatThrownBy()라는 메서드를 제공한다.
assertThat을 사용해서 예외처리를 하려면,
```
    @Test
    public void exception(){
        Throwable thrown = catchThrowable(()-> {throw new Exception("Error");});

        assertThat(thrown).isInstanceOf(Exception.class).hasMessageContaining("Error");
    }
```
이렇게 thrown과 같은 객체를 만들어주어야 하지만, assertThatThrownBy()를 사용하면 
```
class aboutStringTest {
    @Test
    public void exception(){
       assertThatThrownBy(()->{throw new Exception("Error");})
               .isInstanceOf(Exception.class)
               .hasMessageContaining("Error");

    }
}
```
이처럼 가독성 있는 코드를 짤 수 있다.

## String 클래스에 대한 학습 테스트
### 요구사항 1
* "1,2"을 ,로 split 했을 때 1과 2로 잘 분리되는지 확인하는 학습 테스트를 구현한다.
* "1"을 ,로 split 했을 때 1만을 포함하는 배열이 반환되는지에 대한 학습 테스트를 구현한다.

힌트       
* 배열로 반환하는 값의 경우 assertj의 contains()를 활용해 반환 값이 맞는지 검증한다.
* 배열로 반환하는 값의 경우 assertj의 containsExactly()를 활용해 반환 값이 맞는지 검증한다.


```
class aboutStringTest {
    @Test
    void split(){
        String[] actual = "1,2".split(",");
        String[] actual2 = "1".split(",");

        assertThat(actual).contains("1","2");
        assertThat(actual2).contains("1");
    }

}

```

### 요구사항 2

```
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;
import static org.assertj.core.api.Assertions.*;

class aboutStringTest {
    @Test
    void split(){
        String actual = "(1,2)".substring(1,4);

        assertThat(actual).isEqualTo("1,2");

    }

}

```

### 요구사항 3
* "abc" 값이 주어졌을 때 String의 charAt() 메소드를 활용해 특정 위치의 문자를 가져오는 학습 테스트를 구현한다.
* String의 charAt() 메소드를 활용해 특정 위치의 문자를 가져올 때 위치 값을 벗어나면 StringIndexOutOfBoundsException이 발생하는 부분에 대한 학습 테스트를 구현한다.
* JUnit의 @DisplayName을 활용해 테스트 메소드의 의도를 드러낸다.

```
class aboutStringTest {
    @Test
    @DisplayName("charAt()이 특정 문자의 위치를 정확히 가져오는지 확인하는 코드")
    public void exception(){
       String actual = "abc";

       assertThat(actual.charAt(0)).isEqualTo('a');
       assertThat(actual.charAt(1)).isEqualTo('b');
       assertThat(actual.charAt(2)).isEqualTo('c');

       assertThatExceptionOfType(IndexOutOfBoundsException.class)
               .isThrownBy(()-> {
                   char val = actual.charAt(3);
                   throw new Exception("범위초과");

               }).withMessageContaining("String");
    }
}

```

## Set Collection에 대한 학습 테스트

* 다음과 같은 Set 데이터가 주어졌을 때 요구사항을 만족해야 한다.

```
public class SetTest {
    private Set<Integer> numbers;

    @BeforeEach
    void setUp() {
        numbers = new HashSet<>();
        numbers.add(1);
        numbers.add(1);
        numbers.add(2);
        numbers.add(3);
    }
    
    // Test Case 구현
}
```

### 요구사항 1
* Set의 size() 메소드를 활용해 Set의 크기를 확인하는 학습테스트를 구현한다.

```
  @Test
    @DisplayName("Set의 사이즈 확인")
    void size(){
        int size = numbers.size();
        assertThat(size).isEqualTo(3);
    }
```

### 요구사항 2
* Set의 contains() 메소드를 활용해 1, 2, 3의 값이 존재하는지를 확인하는 학습테스트를 구현하려한다.
* 구현하고 보니 다음과 같이 중복 코드가 계속해서 발생한다.
* JUnit의 ParameterizedTest를 활용해 중복 코드를 제거해 본다.
```
 @ParameterizedTest
    @ValueSource(ints = {1,2,3})
    @DisplayName("Set에 해당 요소가 존재하는지 확인")

    void contains(int input){
        assertTrue(numbers.contains(input));
    }
```

### 요구사항 3
* 요구사항 2는 contains 메소드 결과 값이 true인 경우만 테스트 가능하다. 입력 값에 따라 결과 값이 다른 경우에 대한 테스트도 가능하도록 구현한다.
* 예를 들어 1, 2, 3 값은 contains 메소드 실행결과 true, 4, 5 값을 넣으면 false 가 반환되는 테스트를 하나의 Test Case로 구현한다.
```
    @ParameterizedTest
    @CsvSource(value = {"1: true", "2: true", "3 : true", "4:false","5:false"}, delimiter = ':')
    @DisplayName("입력 값에 따라 결과 값이 다른 경우의 테스트")
    void diffrentReturn(int input, boolean expected){
        assertThat(numbers.contains(input)).isEqualTo(expected);

    }
```
