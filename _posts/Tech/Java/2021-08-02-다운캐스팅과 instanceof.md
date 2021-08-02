---
title : "[Java] 다운 캐스팅과 instanceof"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true
---
## 하위 클래스로 형 변환, 다운캐스팅
앞에서 상위 클래스로 형 변환이 묵시적으로 이루어지는 과정을 알아보왔다. 여기서는 다시 하위 클래스로 형 변환이 되는 과정을 살펴보자.    

Animal 이라는 클래스가 있고, 하위 클래스로  Human, Tiger, Eagle 세가지 클래스가 있다면   
우리는 상위클래스를 자료형으로 선언하는 Animal ani = new Human(); 코드를 사용할 수 있다.
이때 생성된 인스턴스 Human은 Ainimal 형이다. 이렇게 Animal형으로 형 변환이 이루어진 경우에는 Animal 클래스에서 선언한 메서드와 멤버변수만 사용할 수 있다. 따라서 Human 클래스에 더 많은 메서드가 구현되어 있고 다양한 멤버 변수가 있다고 하더라도 자료형이 Animal 이라면 사용할 수가 없을 것 이다. 따라서 다시 원래의 인스턴스 자료형으로 되돌아가야 하는 경우가 있다. 이렇게 상위클래스로 형 변환된 하위 클래스를 원래 자료형으로 형 변환하는 것을 **다운캐스팅(down casting)** 이라고 한다.

## instanceof
상속관계를 생각해보면 모든 인간은 동물이지만, 모든 동물이 인간은 아니다. 따라서 다운캐스팅을 하기 전에 상위 클래스로 형 변환된 인스턴스의 원래 자료형을 확인해야 변환할 때 오류를 막을 수 있다. 이를 확인하는 예약어가 바로 instanceof 인데, instanceof의 사용 방법은 다음과 같다.
```
Animal hAnimal = new Human();
if(hAnimal instanceof Human){
    Human human = (Human)hAnimal
}
```
위에서 사용한 hAnimal은 원래 Human형으로 생성되었는데, Animal형으로 형 변환된다. instanceof 예약어는 왼쪽에 있는 변수의 원래 인스턴스형이 오른쪽 클래스 자료형인가를 확인한다. 코드를 보면 hAnimal이 Animal 형으로 되어있지만, 원래는 Human형으로 생성된 인스턴스인지 확인하는 것이다. instanceof의 반환값이 true이면 다운 캐스팅을 하는데, 이때는 Human human = (Human)hAnimal; 문장과 같이 명시적으로 자료형을 써주어야 한다.
상위 클래스 형변환은 묵시적으로 가능하지만, 하위클래스로 형변환은 명시적으로 해야하기 때문이다.

다음처럼 원래 자료형이 Human이 아닌 경우를 보자.
```
Animal ani = new Tiger();
Human h = (Human)ani;
```
위와 같이 코딩해도 컴파일 오류는 나지 않는다. 왜일까? 일단 Tiger 인스턴스는 상위클래스로의 형 변환이기 때문에 자동으로 형 변환이 된다. 변수 h의 자료형 Human과 강제 형 변환되는 ani(Human)의 자료형이 동일하므로 컴파일 오류는 일어나지 않는다. 그 대신 이 코드를 실행하면 오류가 발생한다.

따라서 참조 변수의 원래 인스턴스형을 정확히 확인하고 다운 캐스팅을 해야 안전하며, 이때 instanceof를 사용하는 것이다. 그럼 원래 인스턴스 형으로 가운 캐스팅하는 예를 살펴보도록 하자.
```
package downcasting;
import java.util.ArrayList;

class Animal {
    public void move(){
        System.out.println("동물이 움직입니다.");
    }

}

class Human extends Animal{
    @Override
    public void move() {
    System.out.println("사람이 걷습니다.");
    }
    public void readBook(){
        System.out.println("사람이 책을 읽습니다.");
    }
}

class Tiger extends Animal{
    @Override
    public void move(){
        System.out.println("호랑이가 네발로 깁니다.");
    }
    public void hunting(){
        System.out.println("호랑이가 사냥을 합니다.");
    }
}

class Eagle extends Animal{
    @Override
    public void move(){
        System.out.println("독수리가 날아갑니다..");
    }
    public void flying(){
        System.out.println("독수리가 멀리 납니다.");
    }
}

public class AnimalTest {
    ArrayList<Animal> aniList = new ArrayList<Animal>();

    public static void main(String[] args){
        AnimalTest aTest = new AnimalTest();
        aTest.addAnimal();
        System.out.println("===다운 캐스팅===");
        aTest.testCasting();
    }
    public void addAnimal(){
        aniList.add(new Human());
        aniList.add(new Tiger());
        aniList.add(new Eagle());

        for(Animal ani : aniList){
            ani.move();
        }
    }
    public void testCasting(){
        for(int i = 0; i < aniList.size();i++){
            Animal ani = aniList.get(i);
            if(ani instanceof Human){
                Human h = (Human)ani;
                h.readBook();

            }
            else if(ani instanceof Tiger){
                Tiger t = (Tiger) ani;
                t.hunting();
            }
            else if(ani instanceof Eagle){
                Eagle e = (Eagle) ani;
                e.flying();
            }
            else{
                System.out.println("지원되지 않는 자료형입니다.");
            }
        }

    }
}


>>> 사람이 걷습니다.
>>> 호랑이가 네발로 깁니다.
>>> 독수리가 날아갑니다..
>>> ===다운 캐스팅===
>>> 사람이 책을 읽습니다.
>>> 호랑이가 사냥을 합니다.
>>> 독수리가 멀리 납니다.

```
위의 코드를 보면 각 동물 클래스를 인스턴스로 생성하여 Animal 형으로 선언한 배열에 추가한다. 이렇게 되면 배열에 추가되는 요소의 자료형은 모두 Animal 형으로 변환되는데, 이때 호출할 수 있는 메서드는 Animal 클래스에 선언된 메서드 뿐이다. 이렇게 선언된 배열에서 요소를 하나씩 꺼내 move() 메서드를 호출 하면 제정의한 메서드가 호출된다.
하지만 배열요소가 Animal 형이므로, 각각에 있는 메서드인 readBook(), hunting(), flying() 메서드는 사용할 수 없기 때문에, for 문을 사용하여 각각을 if문으로 다운캐스팅 해주어야만 각각의 메서드를 실행 시킬 수 있다.

