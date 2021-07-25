---
title: "[Programmers] 캐시"
toc : true
toc_sticky : true
toc_label : "On this Page"
categories : programmers

---
## 문제 
지도개발팀에서 근무하는 제이지는 지도에서 도시 이름을 검색하면 해당 도시와 관련된 맛집 게시물들을 데이터베이스에서 읽어 보여주는 서비스를 개발하고 있다.
이 프로그램의 테스팅 업무를 담당하고 있는 어피치는 서비스를 오픈하기 전 각 로직에 대한 성능 측정을 수행하였는데, 제이지가 작성한 부분 중 데이터베이스에서 게시물을 가져오는 부분의 실행시간이 너무 오래 걸린다는 것을 알게 되었다. 

어피치는 제이지에게 해당 로직을 개선하라고 닦달하기 시작하였고, 제이지는 DB 캐시를 적용하여 성능 개선을 시도하고 있지만 캐시 크기를 얼마로 해야 효율적인지 몰라 난감한 상황이다.

어피치에게 시달리는 제이지를 도와, DB 캐시를 적용할 때 캐시 크기에 따른 실행시간 측정 프로그램을 작성하시오.

## 입력 형식
캐시 크기(cacheSize)와 도시이름 배열(cities)을 입력받는다.
cacheSize는 정수이며, 범위는 0 ≦ cacheSize ≦ 30 이다.
cities는 도시 이름으로 이뤄진 문자열 배열로, 최대 도시 수는 100,000개이다.
각 도시 이름은 공백, 숫자, 특수문자 등이 없는 영문자로 구성되며, 대소문자 구분을 하지 않는다. 도시 이름은 최대 20자로 이루어져 있다.

## 출력 형식
입력된 도시이름 배열을 순서대로 처리할 때, "총 실행시간"을 출력한다.

## 조건
캐시 교체 알고리즘은 LRU(Least Recently Used)를 사용한다.
cache hit일 경우 실행시간은 1이다.
cache miss일 경우 실행시간은 5이다.
## 입출력 예제
|캐시크기(cacheSize)|도시이름(cities)|실행시간|
|:---:|:---:|:---:|
3|["Jeju", "Pangyo", "Seoul", "NewYork", "LA", "Jeju", "Pangyo", "Seoul", "NewYork", "LA"]|50
3|["Jeju", "Pangyo", "Seoul", "Jeju", "Pangyo", "Seoul", "Jeju", "Pangyo", "Seoul"]	|21
2|["Jeju", "Pangyo", "Seoul", "NewYork", "LA", "SanFrancisco", "Seoul", "Rome", "Paris", "Jeju", "NewYork", "Rome"]|60
5|["Jeju", "Pangyo", "Seoul", "NewYork", "LA", "SanFrancisco", "Seoul", "Rome", "Paris", "Jeju", "NewYork", "Rome"]|52
2|["Jeju", "Pangyo", "NewYork", "newyork"]|16
0|["Jeju", "Pangyo", "Seoul", "NewYork", "LA"]|25

## 문제 풀이
이 문제는 캐시메모리를 파이썬으로 간단하게 구현해보는 것인데, 카카오 문제라 그런지 한번도 풀어본 적이 없는 신유형 느낌이라 당황스러웠다. 우선 이 문제는 캐시와 LRU를 알지 못하면 아예 풀수가 없어서 이에 대해 알아야 풀 수 있다. 

캐시는 메인 메모리와 CPU간의 데이터 속도 향상을 위한 중간 버퍼 역할을 하는 메모리로써, 잠시 저장해둔다는 의미를 갖고, 실제로 이 기능을 한다. 이 문제는 캐시DB를 간단하게 구현해보는 방식인데, 위 문제에서 나온 Cache Hit은 캐시에 이미 값이 저장 되어있는 값을 참조할때를 말하고, Cache Miss는 캐시메모리에 저장이 되어있지 않아서 메인 메모리로 가서 값을 캐시메모리로 저장해야 할 때를 말한다.

LRU는 캐시교체 알고리즘으로, 가장 오래전에 참조된 값을 삭제하고 새로운 데이터를 그 자리에 삽입하는 알고리즘이다. LRU를 검색하다가 캐시 교체 알고리즘에 대해 알게 되었는데, 내가 이미 큐에서 배웠던 FIFO 방식도 캐시메모리에서 비롯된 알고리즘이란걸 알게되었다.
간단하게 캐시 알고리즘을 정리 해보면 아래의 표와 같다.

|캐시 알고리즘|특징|단점|
|:---:|:---:|:---:|
|Random|교체될 페이지를 임의 선정함|오버헤드가 적음|
|FIFO(First in First Out)|캐시 내에 가장 오래있었던 페이지를 교체|자주사용되는 페이지가 교체될 우려가 있음|
|LFU(Least Frequently Used)|사용 횟수가 가장 적은 페이지부터 교체|최근 적재된 페이지가 교체될 우려가 있음|
|LRU(Least Recently Used)|가장 오랫동안 사용되지 않은 페이지부터 교체|time stamping에 의한 오버헤드 존재|
|Optimal|향후 가장 참조되지 않을 페이지부터 교체|참조될 것을 미리 알 수 없기 떄문에 실현 불가능|
|NUR(Not Used Recently)|참조 비트와 수정 비트로 미사용 페이지 교체, 최근 사용되지 않은 페이지 교체||
|SCR(Second Chance Replacement)|최초 참조 비트 1로 셋팅, 1인 경우 0으로 세팅하고, 0인 경우 교체, 기회를 한번 더 준다.||

따라서 LRU 방식으로 캐시 교체를 하려면, 캐시라는 리스트를 하나 만들어주고 만약 이미 캐시 리스트 안에 있는 값이라면 참조 후 가장 최근위치로 변경해주고 실행시간 1추가, 만약 캐시 리스트 안에 없는 값이라면, 가장 앞에있는 값을 삭제 후 새로운 값을 넣어준 뒤 실행시간 5를 추가 해주면 된다.

주의할점은, 대소문자를 구분하지 않는다고 했기 때문에 반드시 .lower로 소문자로 통일해주어야 하며, 캐시사이즈가 0인 경우는 모두다 Cache Miss이기 때문에 예외처리를 해주어야한다.



```python
def solution(cacheSize, cities):
    #캐시리스트
    cache = []
    #실행시간
    time = 0
    #만약 캐시사이즈가 0인경우는 모두 miss처리한다.
    if cacheSize == 0:
        return len(cities)*5
    #캐시 메모리에 리스트 요소 하나씩 추가
    for city in cities:
        #캐시메모리가 꽉찼을 경우에만 미스와 hit처리
        if len(cache) == cacheSize:
            #hit의 경우
            if city.lower() in cache:
                cache.remove(city.lower())
                cache.append(city.lower())
                time += 1
                #print('hit:',cache)
            #miss의 경우
            else:
                cache.remove(cache[0])
                cache.append(city.lower())
                time+=5
                #print('miss:',cache)
        #캐시메모리의 자리가 있다면 miss인 경우에도 삭제는 필요없음
        else:
            #hit의 경우
            if city.lower() in cache:
                cache.remove(city.lower())
                cache.append(city.lower())
                time += 1
                #print('hit:',cache)
            #miss의 경우
            else:
                cache.append(city.lower())
                time+=5
                #print('miss:',cache)
    return time
```


```python
solution(3,["Jeju", "Pangyo", "Seoul", "Jeju", "Pangyo", "Seoul", "Jeju", "Pangyo", "Seoul"])
```

    firstmiss: ['jeju']
    firstmiss: ['jeju', 'pangyo']
    firstmiss: ['jeju', 'pangyo', 'seoul']
    hit: ['pangyo', 'seoul', 'jeju']
    hit: ['seoul', 'jeju', 'pangyo']
    hit: ['jeju', 'pangyo', 'seoul']
    hit: ['pangyo', 'seoul', 'jeju']
    hit: ['seoul', 'jeju', 'pangyo']
    hit: ['jeju', 'pangyo', 'seoul']





    21



## 노트
프로그래머스에서 다른 풀이를 보니 너무 예쁘게 잘 푼 풀이가 있어서 첨부한다.


```python
def solution(cacheSize, cities):
    import collections
    cache = collections.deque(maxlen=cacheSize)
    time = 0
    for i in cities:
        s = i.lower()
        if s in cache:
            cache.remove(s)
            cache.append(s)
            time += 1
        else:
            cache.append(s)
            time += 5
    return time
```

위 풀이는 deque의 maxlen을 이용했는데, maxlen은 deque의 크기를 제한해주는 메서드로, 그 크기를 넘어서는 값이 들어오면 알아서 가장 앞에있는 값을 삭제하고 새로운 값을 채운다. 이를 사용하면 훨씬 더 파이써닉 하게 풀 수 있을 것 같다.
