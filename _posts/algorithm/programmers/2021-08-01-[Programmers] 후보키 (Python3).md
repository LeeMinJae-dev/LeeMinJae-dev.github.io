---
title : "[Programmers] 후보키 (Python3)"
toc: true
toc_sticky : true
toc_lable : "On this Page"
categories : programmers
---

## 문제 
프렌즈대학교 컴퓨터공학과 조교인 제이지는 네오 학과장님의 지시로, 학생들의 인적사항을 정리하는 업무를 담당하게 되었다.     
그의 학부 시절 프로그래밍 경험을 되살려, 모든 인적사항을 데이터베이스에 넣기로 하였고, 이를 위해 정리를 하던 중에 후보키(Candidate Key)에 대한 고민이 필요하게 되었다.
후보키에 대한 내용이 잘 기억나지 않던 제이지는, 정확한 내용을 파악하기 위해 데이터베이스 관련 서적을 확인하여 아래와 같은 내용을 확인하였다.   

* 관계 데이터베이스에서 릴레이션(Relation)의 튜플(Tuple)을 유일하게 식별할 수 있는 속성(Attribute) 또는 속성의 집합 중, 다음 두 성질을 만족하는 것을 후보 키(Candidate Key)라고 한다.   
 * 유일성(uniqueness) : 릴레이션에 있는 모든 튜플에 대해 유일하게 식별되어야 한다.
 * 최소성(minimality) : 유일성을 가진 키를 구성하는 속성(Attribute) 중 하나라도 제외하는 경우 유일성이 깨지는 것을 의미한다. 즉, 릴레이션의 모든 튜플을 유일하게 식별하는 데 꼭 필요한 속성들로만 구성되어야 한다.   
제이지를 위해, 아래와 같은 학생들의 인적사항이 주어졌을 때, 후보 키의 최대 개수를 구하라.   

|학번| 이름|전공|학년|
|:---:|:---:|:---:|:---:|
|100|ryan|music|2|
|200|apeach|math|2|
|300|tube|computer|3|
|400|con|computer|1|
|500|muzi|music|3|
|600|apeach|music|2|
 
위의 예를 설명하면, 학생의 인적사항 릴레이션에서 모든 학생은 각자 유일한 "학번"을 가지고 있다. 따라서 "학번"은 릴레이션의 후보 키가 될 수 있다.     
그다음 "이름"에 대해서는 같은 이름("apeach")을 사용하는 학생이 있기 때문에, "이름"은 후보 키가 될 수 없다. 그러나, 만약 ["이름", "전공"]을 함께 사용한다면 릴레이션의 모든 튜플을 유일하게 식별 가능하므로 후보 키가 될 수 있게 된다.   
물론 ["이름", "전공", "학년"]을 함께 사용해도 릴레이션의 모든 튜플을 유일하게 식별할 수 있지만, 최소성을 만족하지 못하기 때문에 후보 키가 될 수 없다.   
따라서, 위의 학생 인적사항의 후보키는 "학번", ["이름", "전공"] 두 개가 된다.
릴레이션을 나타내는 문자열 배열 relation이 매개변수로 주어질 때, 이 릴레이션에서 후보 키의 개수를 return 하도록 solution 함수를 완성하라.   
## 제한사항
* relation은 2차원 문자열 배열이다.    
* relation의 컬럼(column)의 길이는 1 이상 8 이하이며, 각각의 컬럼은 릴레이션의 속성을 나타낸다.
* relation의 로우(row)의 길이는 1 이상 20 이하이며, 각각의 로우는 릴레이션의 튜플을 나타낸다.
* relation의 모든 문자열의 길이는 1 이상 8 이하이며, 알파벳 소문자와 숫자로만 이루어져 있다.
* relation의 모든 튜플은 유일하게 식별 가능하다.(즉, 중복되는 튜플은 없다.)

## 입출력 예
|relation|	result|
|:---:|:---:|
|[["100","ryan","music","2"],["200","apeach","math","2"],["300","tube","computer","3"],["400","con","computer","4"],["500","muzi","music","3"],["600","apeach","music","2"]]|	2|

## 입출력 예 설명
입출력 예 #1
문제에 주어진 릴레이션과 같으며, 후보 키는 2개이다.


```python
from collections import defaultdict
from itertools import combinations

def solution(relation):
    N = len(relation[0])
    key_idx = list(range(N))
    candidate_key = []
    
    for i in range(1,N+1):
        for comb in combinations(key_idx,i):
            hist = []
            for rel in relation:
                current_key = [rel[c] for c in comb]
                if current_key in hist:
                    break
                else:
                    hist.append(current_key)
            else:
                for ck in candidate_key:
                    if set(ck).issubset(set(comb)):
                        break
                else:
                    candidate_key.append(comb)
       
    return len(candidate_key) 
```


```python
solution([["100","ryan","music","2"],["200","apeach","math","2"],["300","tube","computer","3"],["400","con","computer","4"],["500","muzi","music","3"],["600","apeach","music","2"]])
```




    2



## 노트
이 문제는 집합형 자료형 set에 대해 알아야 효율적으로 풀이가 가능한 문제다. 사실 중복을 처리할때 뺴고는 set 자료형의 교집합이나 합집합 기능을 사용할일 이 적은데, 미리 알아두면 다른 문제를 풀때도 응용이 가능할 것 같아 정리해두려 한다.

### set 자료형 
set 자료형은 집합에 관련된 것을 처리 하기 위해 만들어진 자료형으로 set 키워드를 사용하거나 중괄호를 이용하여 표현 할 수 있다. 선언 방식은 아래와 같다.


```python
s1 = set({1,2,3})
s2 = set([1,2,3])
s3 = {1,2,3}
```

세 방법 모두 같은 집합을 만들며 빈 집합을 선언하기 위해서는 아래와 같이 사용한다.


```python
s4 = set()
```

집합의 특징은 다음과 같다.
* set() 키워드 혹은 중괄호를 이용한다.
* 순서가 없다
* 고유한 값을 가진다.
* mutable(=값이 변하는) 객체이다.
* 순서가 없기 때문에 리스트나 튜플에서 사용했던 인덱싱은 불가능하다.

### 교집합


```python
s1 = set([1,2,3,4,5])
s2 = set([4,5,6,7,8])

#교집합 메서드
print(s1.intersection(s2))

#교집합 연산자
print(s1 & s2)
```

    {4, 5}
    {4, 5}


위의 두 메서드와 연산자 모두 교집합을 구할 수 있도록 한다.

### 합집합


```python
s1 = set([1,2,3,4,5])
s2 = set([4,5,6,7,8])

#합집합 메서드 union
print(s1.union(s2))

#합집합 연산자 |
print(s1 | s2)
```

    {1, 2, 3, 4, 5, 6, 7, 8}
    {1, 2, 3, 4, 5, 6, 7, 8}


합집합은 union 메서드나 | 연산자를 이용하여 구할 수 있다.

### 차집합


```python
s1 = set([1,2,3,4,5])
s2 = set([4,5,6,7,8])

#차집합 메서드 difference
print(s1.difference(s2))
print(s2.difference(s1))

#차집합 연산자 -
print(s1 - s2)
print(s2 - s1)

```

    {1, 2, 3}
    {8, 6, 7}
    {1, 2, 3}
    {8, 6, 7}


순서가 상관이 없는 교집합, 합집합과는 다르게 차집합은 순서가 상관이 있다.   
차집합을 구할떄는 difference 메서드를 이용하거나 - 연산자를 이용하여 구할 수 있다.

### 집합이 아예 다른지를 확인하는 경우


```python
s1 = {1,2,3,4}
s2 = {5,6,7,8}
s3 = (1,5,6,7)

print(s1.isdisjoint(s2))
print(s1.isdisjoint(s3))
```

    True
    False


isdisjoint 메서드는 두 집합의 요소가 한개도 동일하지 않은지를 확인하는 메서드이다. 만약 요소가 한개라도 같다면 False를 출력하고, 요소가 모두 다를 경우 True를 출력한다.

### 집합이 부분집합인지를 확인하는 경우


```python
s1 = {1,2,3,4}
s2 = {1,2}

print(s1.issubset(s2))
print(s2.issubset(s1))

```

    False
    True


해당 집합이 메서드 배개변수의 집합의 부분집합인지를 판별하여 bool 자료형으로 출력해준다.
