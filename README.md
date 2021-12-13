**Apple이 지원하는 XIB 연결뿐만 아닌, 이니셜라이저와 nib호출을 통해 UIView를 확장해 XIB 사용이 가능하다.**


## XIB를 이용해서 Custom View 제작

**File Owner에 Custom Class 적용**

1. 생성한 Custom Class를 File Owner에 적용

![스크린샷 2021-12-13 오후 3 57 25](https://user-images.githubusercontent.com/93528918/145775968-b207845e-f922-430a-898d-906861db4c7e.png)![스크린샷 2021-12-13 오후 3 59 41](https://user-images.githubusercontent.com/93528918/145775977-e00f3342-ab1a-4f6e-a16e-7e596a9b1d23.png)



2. XIB를 View로 불러오기

<aside>
👉 `xib`로 구성한 View를 가져오기 위해서 `nib`형태로 불러와서 등록하고,
Custom View에 `addSubView`를 실행

</aside>

> ***.first***

File Owner에는 여러개의 View를 가질 수 있기 때문에, 이 중에서 Interface Builder로 Custom한 Class를 가져와 현재 View에 계층을 한 단계 더 쌓은 구조
> 

```swift
**SquareBoxView**

required init?(coder: NSCoder) {
    super.init(coder: coder)
    loadView()
}

func loadView() {
		let view = UINib(nibName: "SquareBoxView", bundle: nil)
								.instantiate(withOwner: self, options: nil).first as! UIView
		view.frame = bounds // view의 frame을 SquareBoxView의 bounds로 설정
		self.addSubview(view)
}
```

---

### 이니셜라이저

UIView는 **NSCoding 프로토콜**을 상속 받고 있다.

이 프로토콜은 표준 키 기반 아카이브를 통해서 직렬화가능한 Class에 적용되는데, nib 파일에 들어갈 수 있는 모든 클래스는 이 프로토콜을 따르고 있다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3775756c-50c2-4dbb-8025-beb2c92e7d16/Untitled.png)

<aside>
👉 두 initializer의 차이는 Interface Builder로 불러올 때와 코드로 생성할 때, 각각 불린다는 차이가 있다.

</aside>

- **required init?(coder: NSCoder)**

> *Returns an object initialized from data in a given unarchiver (공식문서)*
> 

***unarchive***

스토리보드나 xib를 활용하면 코드를 작성하지 않고 앱의 속성을 수정할 수 있는데, 이것을 가능하게 해주는 과정

Interface builder는 코드가 아니기 때문에 앱을 컴파일 하는 시점에서 컴파일러가 인식할 수 없고, 이를 코드로 변환해주는 unarchiving 과정이 필요하다는 것이다.

이 과정에서 init?(coder:)가 사용된다.

파라미터인 `coder`를 통해 NSCoder 타입의 object가 전달되는 것이다. 전달된 NSCoder 타입의 object가 decoding 되어 초기화된 후 컴파일 할 수 있게 decoding 된 자기자신(self)를 반환하는 작업이라고 볼 수 있다. 

- **override init(frame: CGRect)**

스토리보드, xib, nib과 같은 interface builder를 사용하지 않고, 코드로 UIView class의 View object를 만들기 위해 지정된 이니셜라이저

그래서 코드로 작성하려는 label, button, view 등 frame을 가진 모든 view 객체는 해당 이니셜라이저 안에서 초기화가 되어야한다.

우리가 일반적으로 UIView를 생성하고 Nib으로 생성자를 생성하면 매번 이런 오류가 나오는데,

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f057e3f3-8e89-4873-a437-ee31a875ad99/Untitled.png)

모든 View는 NSCoding을 채택하고 있고, 해당 init(coder:) required이기 때문에

코드로 UI작성을 할 때는 해당 이니셜라이저를 사용하지 않음에도 불구하고, View를 구현할 때 선언해주어야한다.

---

### nib 로드(코드에서 .xib 파일들을 생성하여 참조)

> 컴파일을 하기 위해 nib으로 변환한 뒤 (시스템에서 nib파일을 unarchive), nib파일을 로드
> 

Interface builder는 코드가 아니기 때문에 앱을 컴파일 하는 시점에서 컴파일러가 인식할 수 없고, 이를 코드로 변환해주는 unarchiving 과정이 필요하다.

<aside>
👉 **xib:** **XML Interface Builder (태그 형태의 마크업 언어)**
XCode가 Interface Builder를 통해 시각적으로 제어 가능하도록 제공

****nib: NeXT Interface Builder**
뷰의 layout, display등의 요소들을 object graph로 만들어서 직렬화한 파일

</aside>

- UINib

UINib Class는 nib 파일의 컨텐츠를 래핑하는 객체

UINib 객체는 nib 파일의 컨텐츠(View)를 메모리에 캐시하고 있다가, `instantiate`할 때 unarchiving한다.

```swift
let view = UINib(nibName: "SquareBoxView", bundle: nil)
						.instantiate(withOwner: self, options: nil).first as! UIView
```

- Bundle.main.loadNibNamed

nib 파일을 이름으로 찾아서 메모리에 로드하여, nib파일 내의 top-level 객체들을 [Any]? 타입으로 반환

```swift
let view = Bundle.main.loadNibNamed("SquareBoxView", owner: self, options: nil)
```

---

### @IBInspectable, @IBDesignable

- **@IBInspectable**

![스크린샷 2021-12-13 오전 11.26.27.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/06d1832d-e8c5-4446-80ab-e9d2d27f1ec4/스크린샷_2021-12-13_오전_11.26.27.png)

```swift
@IBInspectable var cornerRadius: CGFloat {
    get {
        return layer.cornerRadius
    }
    set {
        layer.cornerRadius = newValue
    }
}

@IBInspectable var borderWidth: CGFloat {
    get {
        return layer.borderWidth
    }
    set {
        layer.borderWidth = newValue * 2
    }
}

@IBInspectable var borderColor: UIColor {
    get {
        return UIColor(cgColor: layer.borderColor!)
    }
    set {
        layer.borderColor = newValue.cgColor
    }
}
```

- **@IBDesignable**

![스크린샷 2021-12-13 오전 11.28.22.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2e5cc6c1-c57a-480f-9194-c15dba4e0647/스크린샷_2021-12-13_오전_11.28.22.png)

```swift
@IBDesignable
class MainActivateButton: UIButton {
		...
}
```

---

> 참고
> 

[inwoodev.log](https://velog.io/@inwoodev/iOS-initframe-initcoder)

[김종권의 iOS 앱 개발 알아가기](https://ios-development.tistory.com/311)
