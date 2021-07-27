---
title : "[Java] 배열과 ArrayList"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true
---
## 배열 선언과 초기화
배열을 사용하려면 먼저 배열을 선언해야한다.   
배열도 변수와 마찬가지로 자료형을 함께 선언하는데, 배열을 선언하는 문법은 다음과 같다.   
```
자료형[] 배열 이름 = new 자료형[개수];
자료형 배열 이름[] = new 자료형[개수];
```
예를들어 학생의 학번을 만약 배열로 선언한다고 하면,
```
int[] studentIDs = new int[10];
```
이렇게 선언하면 studentIDs라는 이름을 가진 int형 10자리 배열이 생성된다.

만약 선언과 동시에 배열의 값을 초기화하고 싶다면,   
중괄호를 사용하면 배열의 값을 초기화하면서 선언할 수 있다.
```
int[] studentIDs = new int[]{101,102,103}
```
이때, 알아서 초기화 되는 갯수 만큼 배열이 생성되므로, []안의 개수는 생략해야한다.
또 다음과 같이 new int[] 부분을 아예 생략할 수도 있다.
```
int[] studentIDs = {102,102,103}
```
## 배열 사용하기
선언한 배열의 요소에 접근하려면 [ ]를 사용한다.   
만약 배열의 첫번째 요소에 10을 저장한다면 아래처럼 코드를 작성하면된다.
```
studentIDs[0] = 10;
```
배열의 순서는 항상 0부터 시작한다.

그럼 위의 코드를 하나의 프로그램으로 해서 배열을 출력해보도록 하자.
```
package array;

public class ArrayTest {
    public static void main(String[] args){
        int[] num = new int[] {1,2,3,4,5,6,7,8,9,10};

        for (int i = 0;i<num.length;i++){
            System.out.println(num[i]);
        }
    }
}
>>> 1
>>> 2
>>> 3
>>> 4
>>> 5
>>> 6
>>> 7
>>> 8
>>> 9
>>> 10
```
위처럼 배열에 담긴 1부터 10까지의 수가 차례로 출력된것을 볼 수 있다.

자바의 배열은 배열길이를 가지는 legth 속성을 가지는데, 자바에서 배열길이는 처음에 선언한 배열의 전체 요소 개수를 의미한다. 전체 길이를 알고 싶은 배열 이름뒤에 도트 연산자를 붙이고 length 속성을 쓰면 배열 길이를 반환해준다. for문으로 배열의 모든 요소를 출력하고 싶다면, 조건식에 .length를 사용해줌으로써 배열의 길이만큼 출력하도록 할 수 있다. 

그럼 빈 값이 들어가있는 배열에서 값이 있는 요소만 출력하고 싶다면 어떻게 해야 할까?
아래의 코드와 같이 size라는 변수를 하나 더 만들어 사용해주면 된다.
```
package array;

public class ArrayTest {
    public static void main(String[] args){
        int[] num = new int[5];
        int size = 0;
       num[0] = 10; size++;
       num[1] = 20; size++;
       num[2] = 30; size++;

       for(int i = 0;i<size;i++){
           System.out.println(num[i]);
       }
    }
}
```

## 객체 배열 사용하기
이번에는 참조 자료형으로 선언하는 객체 배열에 대해 알아보자.   
동일한 기본 자료형 변수 여러개를 사용할 수 있듯이 참조자료형 변수도 여러 개를 배열로 사용 가능하다. 책과 책 저자를 저장하는 클래스 Book을 만들어 객체 배열을 만들어보자.
```
package array;

public class Book {
    private  String bookName;
    private String author;

    public Book(){}
    
    public Book(String bookName, String author){
        this.bookName = bookName;
        this.author = author;
    }
    public String getBookName(){
        return bookName;
    }
    public void setBookName(String bookName){
        this.bookName = bookName;
    }
    public String getAuthor(){
        return author;
    }
    public void setAuthor(String author){
        this.author =author;
    }
    public  void showBookInfo(){
        System.out.println(bookName+","+author);
    }
}
```
Book 클래스를 만들어 책이름과 저자를 저장하는 변수 bookName, author를 만들어주고, 각각을 받아오는 get 함수와 각각을 저장하는 set함수를 만들고, 마지막으로 책이름과 저자를 출력하는 showBookInfo 함수를 만들어주었다.

이제 도서관에 책이 5권있다고 가정하고 Book클래스를 사용하여 책 5권을 객체 배열로 만들어보자.
```
package array;

public class BookArray {
    public static void main(String[] args){
        Book[] library = new Book[5];

        for(int i = 0; i<library.length;i++){
            System.out.println(library[i]);
        }
    }
}

>>> null
>>> null
>>> null
>>> null
```
library = new Book[5]; 를 보면 마치 5개의 인스턴스가 생성된 것처럼 보이지만, 실제로는 인스턴스를 가리키는 주소값을
담을 공간 5개를 생성하는 문장이다. 따라서 이 코드를 실행하게 되면 Book의 주소를 담을 공간 5개가 만들어지고 자동으로 공간이 null로 초기화 된다.

그럼 이제 만들어준 빈공간에 인스턴스를 만들어 저장해보자.
```
package array;

public class BookArray2 {
    public static void main(String[] args){
        Book[] library = new Book[5];
        
        library[0] = new Book("태백산맥", "조정래");
        library[1] = new Book("데미안", "헤르만 헤세");
        library[2] = new Book("1q84", "무라카미 하루키");
        library[3] = new Book("토지", "박경리");
        library[4] = new Book("어린왕자", "생택쥐페리");
    
        for(int i = 0;i<library.length;i++){
            library[i].showBookInfo();
        }
        
    }
}

>>> 태백산맥,조정래
>>> 데미안,헤르만 헤세
>>> 1q84,무라카미 하루키
>>> 토지,박경리
>>> 어린왕자,생택쥐페리
```
출력값을 보면 각 인스턴스가 모두 잘 생성되었음을 알 수 있다.

## 배열 복사하기
기존 배열과 자료형 및 크기가 똑같은 배열을 새로 만들거나 배열의 모든 요소에 자료가 꽉 차서 더 큰 배열을 만들어 
기존 배열에 저장된 자료를 가져오려고 할 때, 배열을 복사한다.   
배열을 복사하는 방법은 두 가지가 있는데, 첫 번째는 기존 배열과 배열 길이가 같거나 더 긴 배열을 만들고 for문을 사용하여 각 요소 값을 반복해서 복사하는 법이고, 두 번째는 System.arraycopy() 메서드를 사용하는 방법이다.

Sysyem.arraycopy(src,srcPos,dest,destPos,length)

각 매개변수의 의미는 다음과 같다.

|매개변수|설명|
|:---:|:---:|
|src|복사할 배열 이름|
|srcPos|복사할 배열의 첫 번째 위치|
|dest|복사해서 붙여 넣을 대상 배열 이름|
|destPos|복사해서 대상 배열에 붙여 넣기를 시작할 첫 번째 위치|
|length|src에서 dest로 자료를 복사할 요소 개수|

그럼 예제를 통해 실제로 배열을 한번 복사해보도록 하자.
```
package array;

public class ArrayCopy {
    public static void main(String[] args){
        int[] array1 = {10,20,30,40,50};
        int[] array2 = {1,2,3,4,5};

        System.arraycopy(array1,0,array2,1,4);
        for(int i = 0;i<array2.length;i++){
            System.out.println(array2[i]);
        }
    }
}
```
src = array1, dest = array2 이므로 array1에 있는 값을 array2에 복사한다.   
srcPos = 0, destPos = 1 이므로, array1의 값은 0부터 array2의 1부터의 공간에 length인 4만큼의 길이로 복사된다.
이때, 복사할 대상 배열의 길이가 복사할 요소 개수보다 작다면 오류가 난다.

이번에는 객체 배열을 복사해보자.   
객체 배열도 마찬가지로 동일한 방식으로 복사해서 사용 할 수 있다.   
```
package array;

public class ObjectCopy1 {
    public static void main(String[] args){
        Book[] library1 = new Book[3];
        Book[] library2 = new Book[3];


        library1[0] = new Book("태백산맥", "조정래");
        library1[1] = new Book("데미안", "헤르만 헤세");
        library1[2] = new Book("1q84", "무라카미 하루키");

        System.arraycopy(library1,0,library2,0,3);

        for(int i=0;i< library2.length;i++){
            library2[i].showBookInfo();
        }
    }
}
>>> 태백산맥,조정래
>>> 데미안,헤르만 헤세
>>> 1q84,무라카미 하루키
```

다음과 같이 library1의 인스턴스가 library2로 복사된 것을 확인 할 수 있다.   
이 상태에서 만약 library1의 값을 변경하게 되면 어떻게 될까??   
예제를 통해 확인해보자.
```
package array;

public class ObjectCopy2 {
        public static void main(String[] args){
            Book[] library1 = new Book[3];
            Book[] library2 = new Book[3];


            library1[0] = new Book("태백산맥", "조정래");
            library1[1] = new Book("데미안", "헤르만 헤세");
            library1[2] = new Book("1q84", "무라카미 하루키");

            System.arraycopy(library1,0,library2,0,3);

            library1[0].setBookName("나목");
            library1[0].setAuthor("박완서");
            
            for(int i=0;i< library2.length;i++){
                library2[i].showBookInfo();
            
        }
    }

}

>>> 나목,박완서
>>> 데미안,헤르만 헤세
>>> 1q84,무라카미 하루키
```
library1[0]의 값을 변경해주었는데, library2[0]의 값도 변경되었다.   
이는 객체 배열의 요소에 저장된 값은 인스턴스 자체가 아니고, 인스턴스의 주소 값이기 때문이다. 따라서 객체 배열을 복사하게되면 인스턴스를 따로 생성하는게 아닌, 기존 인스턴스의 주소값만을 복사하게된다. 결국 두 배열의 서로 다른 요소가 같은 인스턴스를 가리키고 있으므로, 복사되는 배열의 인스턴스 값이 변경되면 두 배열 모두 영향을 받게되는 것이다. 이런 복사를 **얕은 복사(shallow copy)** 라고 한다.

그렇다면 실제로 두 배열의 요소가 각각 다른 주소값의 인스턴스를 가르키도록 복사하려면 어떻게 해야할까?   

이렇게 하고 싶다면, 직접 인스턴스를 만들어 요소 값을 복사해야 한다.    
아래의 코드를 보자.
```
package array;

public class ObjectCopy3 {
    public static void main(String[] args){
        Book[] library1 = new Book[3];
        Book[] library2 = new Book[3];

        library1[0] = new Book("태백산맥","조정래");
        library1[1] = new Book("데미안", "헤르만 헤세");
        library1[2] = new Book("1q84", "무라카미 하루키");

        library2[0] = new Book();
        library2[1] = new Book();
        library2[2] = new Book();
        
        for(int i = 0;i< library2.length;i++){
            library2[i].setBookName(library1[i].getBookName());
            library2[i].setAuthor(library1[i].getAuthor());
        }

        library1[0].setBookName("나목");
        library1[0].setAuthor("박완서");

        for(int i =0;i< library2.length;i++){
            library2[i].showBookInfo();
        }

    }
}
```
위의 코드는 library2의 인스턴스를 직접 만들어 준뒤, setBookName과 setAuthor를 사용하여 하나 하나의 값에 요소값을 복사 해주었다.   
이렇게 하면 복사한 배열 요소는 기존 배열 요소와 서로 다른 인스턴스를 가르키므로 기존 배열의 요소값이 변경되어도 영향을 받지 않는다는 것을 알 수 있다.

## 향상된 for문과 배열
자바 5부터 제공되는 **향상된 for문(enhanced for loop)** 은 배열의 처음에서 끝까지 모든 요소를 참조할 때 사용하면 편리한 반복문으로, 향상된 for문은 배열 요소 값을 하나씩 가져와 변수에 대입한다. 따로 초기화와 종료조건을 명시하지 않아도 알아서 모든 배열의 시작요소부터 끝 요소까지를 실행해준다.
```
package array;

public class EnhancedFor {
    public static void main(String[] args){
        String[] srtArray = {"Java", "Android", "C", "JavaScript", "Python"};

        for(String lang : srtArray){
            System.out.println(lang);
        }
    }
}
```
이처럼 for문의 매개변수에 (String 변수 : 배열 )을 넣어주면 배열의 첫 요소부터 끝까지를 참조해준다.

## ArrayList
우리가 앞서 배운 배열은 기본 프로그램에서 사용하기 위해서는 항상 배열의 길이를 정하고 시작해야 한다.   
하지만 만약 예를 들어 100명의 학생을 위한 프로그램을 개발했는데, 어느 순간 학생 수가 100명이 넘은 경우를 생각해보자. 배열을 사용하는 중에는 배열의 길이를 변경 할 수 없기 때문에 코드를 수정해야한다.
또, 만약 한명이 중간에 전학을 가게되었다면 배열은 중간의 요소를 비워 둘 수 없으므로 배열 요소의 위치를 변경해야한다. 이 두 경우 모두 배열을 하나 하나 수정하려 한다면 매우 힘들고 복잡할 것이다.

그래서 자바는 객체 배열을 더 쉽게 사용 할 수 있도록 객체 배열클래스 ArrayList를 제공한다.    
ArrayList 클래스는 객체 배열을 관리하는 멤버변수와 메서드를 따로 제공하기 때문에, 사용방법만 알아 둔다면 편리하게 사용가능하다.

ArrayList 클래스는 이미 만들어져 있는 메서드가 있는데, 가장 주로 사용하는 메서드를 정리해보자면 다음과 같다.   

|메서드|설명|
|:---:|:---:|
|boolean add(E e)|요소 하나를 배열에 추가한다. E는 요소의 자료형을 의미한다.|
|int size()|배열에 추가된 요소 전체 개수를 반환한다.|
|E get(int index)| 배열의 index 위치에 있는 요소 값을 반환한다.|
|E remove(int index)|배열의 index 위치에 있는 요소 값을 제거하고 그 값을 반환한다.|
|boolean isEmpty()|배열이 비어있는지 확인한다.|

add( ) 메서드를 사용하면 배열 길이와 상관없이 객체를 추가 할 수 있다. 만약 배열 요소 개수가 부족하다면
배열 크기를 더 키울 수 있도록 구현되어 있으며, 또 배열 중간의 어떤 요소가 제거되면 그 다음 요소 값을 하나씩 앞으로 이동하는 코드도 이미 구현되어 있기때문에, 훨씬 편리하게 프로그래밍 할 수 있다.

## ArrayList 활용하기
ArrayList를 사용하는 방법은 다음과 같다.
```
ArrayList<E> 배열 이름 = new ArrayList<E>();
```
선언하는 부분 <>안에 사용할 객체의 자료형을 쓰면 된다.   
예를들어 앞에서 살펴본 Book 클래스형을 자료형으로 사용해서 ArrayList 배열을 생성한다면 다음과 같다.
```
ArrayList<Book> library = new ArrayList<Book>();
```
ArrayList는 java.util 패키지에 구현되어 있는 클래스로, 현재 만든 프로그램에는 이 패키지가 포함되어 있지 않기 때문에 컴파일러에게 ArrayList를 사용하기 위해서는 컴파일러에게 ArrayList가 어디에 구현되어 있다고 알려주기 위해 코드 맨 위에 선언하는 것을 **임포트(import)** 한다고 한다. 즉 ArrayList를 사용하려면 ArrayList를 import해주어야 사용할 수 있다.
```
package array;
import java.util.ArrayList;

public class ArrayListTest {
    public static void main(String[] args){
        ArrayList<Book> library = new ArrayList<Book>();

        library.add(new Book("태백산맥", "조정래"));
        library.add(new Book("데미안", "헤르만 헤세"));
        library.add(new Book("1q84", "무라카미 하루키"));
        library.add(new Book("토지", "방경리"));
        library.add(new Book("어린왕자", "생택쥐페리"));

        for(int i = 0; i<library.size();i++){
            Book book = library.get(i);
            book.showBookInfo();
        }
        System.out.println();

        System.out.println("=== 향상된 for문 사용 ===");
        for(Book book : library){
            book.showBookInfo();
        }

    }
}
```
기본 배열에서는 [ ] 안에 배열 전체 길이를 미리 지정해야 했습니다. 하지만 ArrayList를 생성할 때는 미리 지정할 필요 없이 add( ) 메서드를 사용해 생성자만 호출하면 된다.   

