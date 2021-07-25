---
title: "[Programmers] JadenCase 문자열 만들기 (Python3)"
toc : true
toc_sticky : true
toc_label : "On this Page"
categories : programmers

---
## 문제 설명
JadenCase란 모든 단어의 첫 문자가 대문자이고, 그 외의 알파벳은 소문자인 문자열입니다. 문자열 s가 주어졌을 때, s를 JadenCase로 바꾼 문자열을 리턴하는 함수, solution을 완성해주세요.

## 제한 조건
s는 길이 1 이상인 문자열입니다.
s는 알파벳과 공백문자(" ")로 이루어져 있습니다.
첫 문자가 영문이 아닐때에는 이어지는 영문은 소문자로 씁니다. ( 첫번째 입출력 예 참고 )

## 입출력 예
```
s	                     return
"3people unFollowed me"	 "3people Unfollowed Me"
"for the last week"	     "For The Last Week"
```
## 코드


```python
s = "my NAMe   4s    3injae"
```


```python
def solution(s):
    s = s.lower().split()
    result = []
    for i in s:
        i = list(i)
        print(i)
        i[0] = i[0].upper()
        result.append("".join(i))
        
    return " ".join(result)
    
```


```python
solution(s)
```




    'My Name   4s    3injae'




```python
s.lower().split()
```




    ['my', 'name', 'is', 'minjae']


