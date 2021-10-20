---
title : "[SwiftUI] ignoreSafeArea(.keyboard)가 이미지에 작동하지 않을 때"
categories : swiftUI
toc : true
toc_label : "On this Page"
toc_stciky : true
---
## 문제
 키보드를 사용하려고 하면 화면 전체가 말려올라가서, 화면이 깨질 뿐더러, 밑에 식단 버튼이 올라오지 않았다.
 
![develop](/assets/images/Tech/SwiftUI/ignoreSafe/Image1.PNG)

따라서 엔터를 눌러 식단추가 버튼이 보이도록 해야 했고, 텍스트필드도 위로 올라가서 자신이 무슨 검색어를 입력하는지
보이지 않았다. 

## 해결방법
맨 처음에는 별 대수롭지 않게 ignoreSafeArea(.keyboard)로 간단하게 해결하려고 했는데, 생각해보니까 텍스트필드가 키보드를 
가리지도 않는데 텍스트필드에 포커스를 주면 화면 전체가 위로 말려 올라가기 때문에 safeArea와는 상관이 없는 오류였다. 
혹시 ignoreSafeArea 코드의 위치가 다른가? 최대한 상위뷰에 써주어야 작동하나? 라는 생각에 계속 ignoreSafeArea자체가 작동 
안하는거에 대한 내용으로 검색을 했다. 하지만 계속 앱을 실행시키다보니 아래 버튼에는 ignoreSafeArea가 작동하여 아래의 버튼은 키보드가 동작해도 
올라오지 않는것을 발견했다. 생각해보니 텍스트필드가 가려져있지 않음에도 올라가는 것도 이상하고 그래서 자세히 보니 추가된 이미지가 정확히 키보드위로 올라와 있었고,
키보드가 올라오면서 위의 뷰들을 밀어내어 전체적으로 올라가고 있던 것이었다. 

따라서 생각을 바꿔 이미지가 키보드에 가려지지 않도록 텍스트 필드에 포커싱이 가면 이를 detecting해서 이미지 사이즈 자체가 줄어들도록 해주었다. 
이를 위해서 텍스트필드에 포커스가 갔는지 안갔는지 확인 하는 코드가 필요했는데, 간단하게 StackOverFlow에서 찾을 수 있었다.

```
@State var text : String = false
TextField("텍스트 필드", text: $text, onEditingChanged: { (editingChanged) in
    if editingChanged {
       
        ///Focus On 인 상태
    } else {
       ///Focus Out 인 상태
        
    }})
                            
``` 

따라서 이를 통해 이미지 자체의 사이즈를 줄였는데, 

![develop](/assets/images/Tech/SwiftUI/ignoreSafe/Image2.PNG)


내가 원하는건 키보드 거리 만큼 이미지는 그대로 사이즈만 줄어들어 자연스럽게 움직이는것이었는데, 그냥 이미지 사이즈가 줄은대로 왜곡되어
이미지가 찌부되는 현상이 일어났다. 이미지는 그대로 있고 잘리듯이 줄어들게 해주는 코드가 필요했다.

간단하게 아래 코드로 이미지의 손상없이 width 값을 줄여줄 수 있었다.
```
Image("image")
    .aspectRatio(contentMode: .fill)
    .frame(width: 360, height: isFocusOn ? 150 : 280, alignment: .center)
    .cornerRadius(15)
```

이미지에 줄수 있는 비율 옵션은 세가지가 있는데 다음과 같다.

- ScaleToFill (기본값)
Image를 정해진 frame에 늘리고 줄여서 우겨넣는다. 애초에 디자인 가이드에 맞게 디자인된 이미지가 아니라면 부적합하다.

- AspectToFill
원본의 비율은 유지한다. 하지만 frame을 이미지의 비율에 맞지않게 넣어주면 프레임밖의 이미지는 잘라버린다.
ImageView 내부에 절대 여백이 생기지 않는다.

- AspectToFit
원본의 비율을 유지한다. Image는 절대 ImageView 바깥으로 나가지않는다.
따라서 비율이 맞지않는 부분은 여백으로 남겨두게 된다.
