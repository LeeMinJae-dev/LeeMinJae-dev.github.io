---
title: "[Spring] delete요청 보내도 삭제 안될때"
categories: java
toc: true
toc_label: "On this page"
toc_sticky: true
---
## 서론
프로젝트를 진행 중에, delete요청을 보냈는데도, 데이터베이스에서 해당 컬럼이 삭제되지 않는 걸 발견 했습니다.

봉사를 불러오면, 해당 봉사의 참가자를 함께 불러오게 하기 위해서
참가자리스트의 엔티티인 ApplyList에 봉사의 엔티티인 Volunteer와 유저의 엔티티인 User를 각각 다대일 맵핑해주고, Volunteer에 일대다로 ApplyList를 맵핑해주었습니다.


### Volunteer
```
public class Volunteer {
    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    @JsonBackReference
    @OneToMany(mappedBy = "volunteer", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Comment> comments = new ArrayList<>();
}
```

### ApplyList
```
public class ApplyList extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn
    private User user;

    @ManyToOne
    @JoinColumn
    private Volunteer volunteer;
}
```

이 상태에서 서비스레이어에서 유저와 봉사를 파라미터로 받아 해당 조건에 맞는 데이터를 삭제하는 deleteByUserAndVolunteer가 제대로 데이터를 삭제하지 못하였습니다.

### VolunteerService
```
  applyListRepository.deleteByUserAndVolunteer(user.get(0), volunteer);
```

## 해결 방법
Volunteer에서 applyList에 있는 cascade 옵션을 삭제해주면 됩니다.

cascade 옵션을 사용할때에는 반드시 등록과 삭제등 두 엔티티의 라이프 사이클이 동일하고, 여러 엔티티가 아닌 단일 엔티티에 종속적일때만 사용해야 합니다.

지금은 Volunteer에서 ApplyList에 영속성 전이 옵션인 cascade를 설정해주었기 때문에, 부모객체인 Volunteer가 전적으로 ApplyList의 생명주기를 관리하게 됩니다.

따라서 ApplyList에 Delete요청을 보내도, 이 객체의 생명주기는 온전히 Volunteer가 관리하기 때문에 삭제가 되지 않았던 것 입니다.

이렇게 cascade 옵션을 사용할때에는 이 두 객체가 같은 생명주기를 갖는것이 서비스 로직상 맞는지 꼭 확인 후 사용해야겠습니다.
