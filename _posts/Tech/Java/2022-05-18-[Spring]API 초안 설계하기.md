---
title: "[Spring] API 초안 설계하기"
categories: java
toc: true
toc_label: "On this page"
toc_sticky: true
---
# 서론
그 동안은 개발하면서 필요할때마다 그때 그때 필요한 API를 만들곤 했는데, 그렇게 진행하면 나중에 API가 서로 중복되서 꼬이는 상황도 발생하기도 하고, 프론트엔드 개발자도 미리 이러한 API를 만들 것이다~ 라는 가이드라인이 있으면 개발이 훨씬 수월하다고 해서,

이번에는 대략적인 API 설계를 초기에 진행하여 문서로써 초안을 제작한 뒤, 이에 맞춰 개발해보려한다.

문서작성 툴은 앞서 게시물에서 다뤘듯이 **GitBook**을 사용하기로 했다.

# API 설계
대략적으로 필요한 도메인은 다음과 같다.

* Volunteer (봉사 활동)
* User (유저)
* Post (게시물)
* Comment (댓글)
* Application (지원서)

## Volunteer
### GET
```
/volunteers
```
현재 데이터 베이스에 있는 모든 봉사들을 받아온다. 

```
/volunteers?pageno = {pageNumber}
```
현재 데이터베이스에 있는 페이지를 10개씩 가져온다.
예를들어, pageNumber가 1이라면, 1번 부터 10번까지의 게시물을 가져온다.

```
/volunteers/{id}
```
해당 아이디에 해당하는 봉사 활동을 가져온다.

### POST
```
/volunteers
```
봉사 활동을 생성한다.

### PATCH
```
/volunteers/{id}
```
해당 아이디에 해당하는 봉사 활동을 업데이트한다.

### DELETE
```
/volunteers/{id}
```
해당 아이디에 해당하는 봉사 활동을 삭제한다.

## User
### GET
```
/users
```
전체 유저들을 가져온다.

```
/users?pageno = {pagaNumber}
```
현재 데이터베이스에 있는 유저들을 10개씩 가져온다.
예를들어, pageNumber가 1이라면, 1번 부터 10번까지의 게시물을 가져온다.

```
users/{id}
```
해당 아이디에 해당하는 유저를 가져온다.

### POST
```
/users
```
새로운 유저를 생성한다.

### PATCH
```
users/{id}
```
해당 아이디의 유저의 정보를 업데이트 한다.

### DELETE
```
/users/{id}
```
해당 아이디에 해당하는 유저를 삭제한다.

## Post
### GET
```
/posts
```
전체 게시물을 가져온다.

```
/posts?pageno = {pagaNumber}
```
현재 데이터베이스에 있는 게시물들을 10개씩 가져온다.
예를들어, pageNumber가 1이라면, 1번 부터 10번까지의 게시물을 가져온다.

```
posts/{id}
```
해당 아이디에 해당하는 게시물을 가져온다.

### POST
```
/posts
```
새로운 게시물을 생성한다.

### PATCH
```
posts/{id}
```
해당 아이디의 게시물의 정보를 업데이트 한다.

### DELETE
```
/posts/{id}
```
해당 아이디에 해당하는 게시물을 삭제한다.

## Application
### GET
```
/applications
```
전체 신청서를 가져온다.

```
/applications?pageno = {pagaNumber}
```
현재 데이터베이스에 있는 신청서들을 10개씩 가져온다.
예를들어, pageNumber가 1이라면, 1번 부터 10번까지의 게시물을 가져온다.

```
applications/{id}
```
해당 아이디에 해당하는 신청서를 가져온다.

### POST
```
/applications
```
새로운 신청서를 등록한다.

### PATCH
```
applications/{id}
```
해당 아이디의 게시물의 신청서를 업데이트 한다.

### DELETE
```
/applications/{id}
```
해당 아이디에 해당하는 신청서를 삭제한다.



이렇게 간단하게 작성한 문서를 바탕으로 GitBook으로 옮겨 작성해 보도록 하자.

## GitBook에 문서 만들기

GitBook의 양식에 맞게 위의 내용을 정리해보자.

GitBook은 별다른 설정 없이도 간단하게 API 문서를 만들도록 도와 준다.

아래 링크로 가면 이 내용에 관한 API 문서 초안을 확인 할 수 있다.

[https://lee-min-jae.gitbook.io/api/reference/api-reference/user](https://lee-min-jae.gitbook.io/api/reference/api-reference/user)

# 마무리하며...
이렇게 간단하게 본격적인 개발 전 API 설계를 끝마쳤다. 개발하며 세세하게 json 응답 예시도 추가해주고, 디테일한 부분도 수정해가면서 문서와 함께 개발을 진행하다보면 문서가 더욱 더 풍성해 질 것 같다.


