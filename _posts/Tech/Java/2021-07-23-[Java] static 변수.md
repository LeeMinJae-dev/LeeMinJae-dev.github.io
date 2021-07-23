---
title : "[Java] static 변수"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true
---
## 변수를 여러 클래스에서 공통으로 사용하기
우리가 앞서만든 학생클래스에서, 만약 학번을 만들고 싶다면 어떻게 해야할까?    
이 학번은 새로운 인스턴스가 생성될때마다 자동으로 생성되어 그 인스턴스에 할당되게 하고싶다.   
이렇게 하려면 인스턴스마다 따로 변수가 생성되는게 아니라, 클래스 전체가 함께 사용하는 기준 변수가 하나 있어야한다. 이러한 변수가 있다면, 우리는 학번을 1씩 증가시켜가며 새로운 인스턴스가 생길때마다 할당 해주면 학번이 자동으로 정해지도록 할 수 있을 것이다. 

이때, 클래스에서 사용하는 공통 변수를 **static 변수**로 선언한다.

## static 변수의 정의와 사용 방법
static 변수는 다른 용어로 '정적 변수'라고도 한다. static 변수는 자바뿐만 아니라 다른 언어에서도 비슷한 개념으로 사용하는 변수로, 다른 멤버 변수와 동일하게 내부에 선언한다.   
자료형앞에 static 예약어를 선언해주면 사용 할 수 있다.
```
static int seralNum;
```
static 변수는 클래스 내부에 선언하지만, 다른 멤버 변수처럼 인스턴스가 생성될 때마다 새로 생성되는 변수가 아니라, 프로그램이 실행되어 메모리에 올라가면 딱 한 번 메모리 공간이 할당되고, 그 값을 모든 인스턴스가 공유하는 변수이다.

그럼 이러한 static변수를 갖고 학번을 가지는 학생 클래스를 만들어 보자.
```
package staticex;

public class Student {
    static int serialNum = 1000;
    String studentName;
    int studentID;
    
    public Stirng getStudentName(){
        return studentName
    }
    public void setStudentName(String name){
        studentName = name;
        
    }

}
```
학생 클래스는 serialNum이라는 static 변수를 갖는데, 이 값은 1000이다.
이제 테스트 코드에서 이 static 변수를 증가시키면, 이 값을 다른 인스턴스들이 공유하는지 한번 확인해보자.

```
package staticex;

public class StudentTest {
    public static void main(String[] args){
        Student Lee = new Student();
        Lee.setStudentName("Lee");
        System.out.println(Lee.serialNum);
        Lee.serialNum++;

        Student Kim = new Student();
        Kim.setStudentName("Kim");
        System.out.println(Kim.serialNum);

        System.out.println(Lee.serialNum);
        System.out.println(Kim.serialNum);
    }
}

>>> 1000
>>> 1001
>>> 1001
>>> 1001
```
Lee 학생의 serialNum을 증가시켰는데, Kim학생의 serialNum도 동일하게 증가된 것을 확인 할 수 있다.   
이처럼 static변수의 값은 모든 인스턴스가 공유한다는 것을 알게되었다. 그럼 이제 인스턴스가 생성되면 자동으로 증가한 학번을 할당하도록 해보자.
```
package staticex;

public class Student {
    static int serialNum = 1000;
    String studentName;
    int studentID;

    public Student(){
        serialNum++;
        studentID = serialNum;
    }
    public String getStudentName(){
        return studentName;
    }
    public void setStudentName(String name){
        studentName = name;


    }

}
```
public Student를 만들어 새로운 인스턴스가 생성되면 serialNum을 1증가시키고, 그 값을 학번에다 할당하도록 했다. 그럼 다시 테스트 코드로 가서 학번이 올바르게 증가하며 할당되는지 확인 해보자.
```
package staticex;

public class StudentTest {
    public static void main(String[] args){
        Student Lee = new Student();
        Lee.setStudentName("Lee");
        System.out.println("이름: "+Lee.studentName+" 학번: "+Lee.serialNum);

        Student Kim = new Student();
        Kim.setStudentName("Kim");
        System.out.println("이름: "+Kim.studentName+" 학번: "+Kim.serialNum);
    }
}

>>> 이름: Lee 학번: 1001
>>> 이름: Kim 학번: 1002
```
인스턴스를 생성해줄때마다 자동으로 학번이 증가하는 것을 확인 할 수 있다.

