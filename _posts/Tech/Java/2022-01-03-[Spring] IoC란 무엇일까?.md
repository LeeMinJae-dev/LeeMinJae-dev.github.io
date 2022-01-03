---
title : "[Spring] IoC란 무엇일까?"
categories : java
toc : true
toc_label : "On this Page"
toc_stciky : true

---
스프링을 사용하다보니, 스프링의 특징 중 하나인 IoC, Inversion of Control의 개념이 이해가 잘되지 않아서 정리해보았다.

이 게시물은 유튜브의 백기선님의 강의인 "스프링 입문"을 보고 공부하며 작성한 글입니다.

> 출처
> [스프링 입문 - 5.Inversion of Control](https://www.youtube.com/watch?v=NZ_lPFvu9oU&list=PLfI752FpVCS8_5t29DWnsrL9NudvKDAKY&index=5)

## 일반적인 제어권
일반적으로 자바에서는 의존성에 대한 제어권이 개발자에게 있다. 따라서 내가 사용할 의존성은 내가 만들어야한다.

```
class OwnerContoller {
    private OwnerRepository repository = new OwnerRepository();
}
```
위 와 같이 repository라는 new OwnerRepository(); 처럼 직접 만들어주어야한다.

## IoC
하지만 위와 다르게 아래 코드를 보면,
```
class OwnerController{
    private OwnerRepository repo;
    
    public OwnerController(OwnerRepository repo) {
        this.repo = repo
    }
    
    // repo를 사용한다.
}
```
이렇게 하면 OwnerContoller는 OwenerRepository를 사용은 하지만, 이 객체를 만들지는 않는다. 따라서 외부에서 누가 만들어준 OwnerRepository를 받아와야 사용할 수 있으므로, 의존성을 외부에서 만들어주어야 한다. 그렇기 떄문에 이것은 제어권이 역전되었다고 볼 수 있는데, 이를 Inversion of Control, IoC라고 한다.

스프링은 관리하는 bean으로 등록되어 있다면, 이 의존성을 알아서 타입에 맞도록 가져와서 주입해준다. 그렇기 때문에 제어권을 스프링이 가지므로 제어권이 개발자에게 있는게 아니라 스프링에게 있다고 할 수 있는데, 이렇게 하면 스프링이 알아서 제어권을 갖고 직접 관계를 부여하기 때문에 개발자 입장에서는 훨씬 신경 써야 할 것이 줄어든다.
