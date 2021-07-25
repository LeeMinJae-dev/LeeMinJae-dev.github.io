---
title: "[Programmers] 행렬의 곱셈 (Python3)"
toc : true
toc_sticky : true
toc_label : "On this Page"
categories : programmers

---
## 문제 설명
2차원 행렬 arr1과 arr2를 입력받아, arr1에 arr2를 곱한 결과를 반환하는 함수, solution을 완성해주세요.

## 제한 조건
행렬 arr1, arr2의 행과 열의 길이는 2 이상 100 이하입니다.
행렬 arr1, arr2의 원소는 -10 이상 20 이하인 자연수입니다.
곱할 수 있는 배열만 주어집니다.

## 입출력 예
```
arr1	                   	        
[[1, 4], [3, 2], [4, 1]]	   
[[2, 3, 2], [4, 2, 4], [3, 1, 4]]

arr2
[[3, 3], [3, 3]]
[[5, 4, 3], [2, 4, 1], [3, 1, 1]]

return
[[15, 15], [15, 15], [15, 15]]
[[22, 22, 11], [36, 28, 18], [29, 20, 14]]
```
## 코드


```python
arr1 = [[1, 4]]  
arr2 = [[3],
        [3]]

```


```python
def solution(arr1, arr2):
    
    result = []
    for x1 in range(len(arr1)):
        su2 = []
        for y2 in range(len(arr2[0])):
            su = []
            for y1 in range(len(arr1[0])):
                su.append(arr1[x1][y1]*arr2[y1][y2])
                print(su)
            su2.append(sum(su))
    
        result.append(su2)
            
    return result
```


```python
solution(arr1, arr2)
```

    [3]
    [3, 12]





    [[15]]


