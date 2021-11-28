---
title : "[Java] 플레이그라운드 with TDD, 클린코드 - 문자열 계산기"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
## 요구사항
* 사용자가 입력한 문자열 값에 따라 사칙연산을 수행할 수 있는 계산기를 구현해야 한다.
* 문자열 계산기는 사칙연산의 계산 우선순위가 아닌 입력 값에 따라 계산 순서가 결정된다. 즉, 수학에서는 곱셈, 나눗셈이 덧셈, 뺄셈 보다 먼저 계산해야 하지만 이를 무시한다.
* 예를 들어 "2 + 3 * 4 / 2"와 같은 문자열을 입력할 경우 2 + 3 * 4 / 2 실행 결과인 10을 출력해야 한다.


## 구현
StringCalculator를 객체로 만들어서 안에서 기능을 처리하도록 한다.

구현해야할 기능을 간단하게 정리해보면,

1. 계산기에 입력된 문자열을 받아 공백을 기준으로 나눠서 배열로 반환해준다.
2. String을 Int형으로 바꿔준다.
3. 사칙연산 연산 후 결과 값을 반환해준다.
4. 들어온 문자를 순서대로 계산해준다.
5. 들어온 문자열이 공백인지 아닌지 확인해준다.
6. 계산 결과를 반환해준다.

## 기능 1번
```
public String[] StringToArray(String str) {
        return str.split(" ");
    }
```
split(" ")을 사용하면 " "의 공백을 기준으로 한칸 씩 띄워서 배열의 형태로 반환해준다.

## 기능 2번
```
public int toInt(String str) {
        return Integer.parseInt(str);
    }
```
integer.parseInt() 메서드는 들어오는 String값이 숫자라면 Int로 반환해준다.

## 기능 3번
```
public int calculate(int firstValue, char operator, int secondValue) {
        if (operator == '+') {
            return add(firstValue, secondValue);
        }
        if (operator == '-') {
            return subtract(firstValue, secondValue);
        }
        if (operator == '/') {
            return divide(firstValue, secondValue);
        }
        if (operator == '*') {
            return multiply(firstValue, secondValue);
        }
        throw new RuntimeException();
    }

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }

    public int divide(int a, int b) {
        try {
            return a / b;
        } catch (Exception e) {
            System.out.println("0으로 나눌 수 없습니다.");
        }
        return 0;
    }

    public int multiply(int a, int b) {
        return a * b;
    }
```
기능을 최대한 나누기 위해서 calculate 메소드 안에 있는 연산들을 따로따로 메서드로 구현했다.      
다른 연산들은 예외가 없지만 devide의 경우에는 0으로 나누었을 때 발생하는 예외를 따로 처리 해줘야 하기 때문에, 단순하게 바로 a/b를 리턴할 수 없기 때문에 예외처리까지한 메서드로 처리해주었다.

## 기능 4번
```
public int calculateSeparateString(String[] str) {
        int result = toInt(str[0]);

        for (int i = 0; i < str.length - 2; i += 2) {
            result = calculate(result, str[i + 1].charAt(0), toInt(str[i + 2]));
        }
        return result;
    }
```
하나씩의 원소로 분리되어 배열로 반환된 문자열을 순서대로 두개씩 계산하는 기능이다.        
순서에 상관없이 바로 연산해야 하므로, for 문을 사용해서 문자열의 첫번째 값부터 시작해 연산자에 따라 연산을 달리해가면서
값을 result에 계산해간다. 

## 기능 5번
```
public boolean isBlank(String input) {
        if (input.equals(" ") || input == null)
            return true;
        return false;
    }
```
혹여나 들어오는 값이 " "와 같은 공백이거나 null 값일 때의 예외 처리를 위해 isBlank라는 메서드를 통해서 연산전에 한번 체크해주도록 한다.

## 기능 6번
```
 public int makeResult(String input) {
        if (isBlank(input))
            throw new RuntimeException();
        return calculateSeparateString(StringToArray(input));
    }
```
최종적으로, 계산기에 문자열을 보내면 위의 메서드들을 통해서 바로 결과값을 반환하는 기능을 구현한다.

## 테스트 코드
```
class StringCalculatorTest {
    StringCalculator stringCalculator;

    @BeforeEach
    public void setUp(){
        stringCalculator = new StringCalculator();
    }

    @Test
    public void stringToArrayTest() {
        String actual = "1 + 3";
        assertThat(stringCalculator.StringToArray(actual))
                .contains(String.valueOf('1'))
                .contains(String.valueOf('+'))
                .contains(String.valueOf('3'));
    }

    @Test
    public void makeResultTest() {
        assertEquals(4,
                stringCalculator.makeResult("1 + 3"));
    }

    @Test
    public void isBlankTest(){
        assertEquals(true, stringCalculator.isBlank(" "));
    }


```

원래 같았다면 따로 main함수를 만들어서 기능이 제대로 동작하는지 빌드해가면서 했겠지만, 이번에는 기능만 구현하고 모든 테스트를
테스트 코드에서 동작시켰다. 

이렇게 하니까 확실히 프로그램 자체가 테스트 코드로 더러워지는 일도 없고, 만약 더 큰 프로젝트라도 일일히 main함수를 돌려가지 않으면서 제대로 코드가 짜여지고 있는지 확인 할 수 있겠다라는 생각이 들었다.

그리고 무엇보다 기댓값이 어떤거였는지 헷갈리지도 않고 한번에 테스트 결과를 확인 할 수 있으니 무척 편리했다.

맨처음 배울때는 왜 굳이 귀찮게 테스트코드를 따로 작성하는지 몰랐는데 이제는 왜 테스트 코드 작성이 중요한지 알 수 있게되었다.
