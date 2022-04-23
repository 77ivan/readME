# 쇼핑몰 상품 목록

RxSwift와 MVVM 패턴을 사용한 쇼핑몰 상품 목록 클라이언트 애플리케이션

![화면 기록 2022-04-22 오후 2 52 37](https://user-images.githubusercontent.com/93528918/164646856-7611a0b7-6424-4153-8a2a-e4e3f60f637b.gif)



<br>

## Content

- [TODO](#TODO)
- [Library](#Library)
- [Rest API 핸들링](#Rest-API-핸들링)
- [페이지네이션](#페이지네이션)
- [footerView indicator](#footerView-indicator)
- [Pull Refresh(새로고침)](#Pull-Refresh(새로고침))
- [이미지 캐싱](#이미지-캐싱)

<br>

## TODO

- Rest API 핸들링
- 네비게이션 설계
- 이미지 캐싱
- Pull Refresh
- 로딩 indicator
- 페이지네이션 (footerView indicator)
- 최상단 이동 버튼

<br>

## Library

- [RxSwift, RxCocoa](https://github.com/ReactiveX/RxSwift)
- [RxDataSources](https://github.com/RxSwiftCommunity/RxDataSources)
- [SnapKit](https://github.com/SnapKit/SnapKit)
- [Kingfisher](https://github.com/onevcat/Kingfisher)
- [Moya](https://github.com/Moya/Moya)
- [Then](https://github.com/devxoul/Then)
- [Toast](https://github.com/scalessec/Toast-Swift)
- [AlignedCollectionViewFlowLayout](https://github.com/mischa-hildebrand/AlignedCollectionViewFlowLayout)


<br>

## Rest API 핸들링


[[ShoppingTarget]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Network/Shopping/ShoppingTarget.swift)

**Moya**

- API를 통해 제공할 기능들을 열거형으로 정의한 후, extension으로 확장해서 TargetType 프로토콜을 채택하여 Moya에서 제공하는 기본적인 네트워킹 레이어를 구축 

<br>

[[ShoppingAPI]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Network/Shopping/ShoppingAPI.swift)

**프로토콜 (ShoppingAPIProtocol)**

- 추후에 아키텍처를 **재사용성, 확장성**이 좋게 만들기 위해서 프로토콜 생성

<br>

**Generic 타입을 사용하여 request 코드 재사용**

- 에러 핸들링을 위해 request를 **Single**로 wrapping해서 사용, 응답값 또는 에러를 발행

- 구독 시, success와 error 두 가지 이벤트 처리

<br>

**Observable<T> → Single<T> 리펙토링**
	
- Network Result 에러의 경우, 계속해서 요청을 하는 것이 아닌 error 이벤트를 발행하여 dispose 처리가 되어야한다.
	
- Network Result 자체를 success 이벤트로 발행하여 에러 핸들링과 동시에 구독 관리를 원활하게 할 수 있도록 할 수 있다.

<br>
<br>

[[ShopingMallViewModel]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Presentation/TabBar/ShoppingMall/ShopingMallViewModel.swift)

- 상품 목록

**Relay** : completed, error를 발생하지 않고 Dispose되기 전까지 계속 작동하기 때문에 UI Event에서 사용

**BehaviorRelay**: 기존 값을 유지하면서 새로운 값을 accept

```swift
var shoppingList = BehaviorRelay<[Product]>(value: [])

...
func populateShoppingProducts(offset: Int) {
    shoppingAPI.populateShoppingProducts(offset: offset)
        .subscribe(onNext: { products in
            self.shoppingList.accept(products.products) /// 새로운 값을 accept
        })
        .disposed(by: disposeBag)
}
```

<br>

## 페이지네이션

[[ShopingMallViewModel]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Presentation/TabBar/ShoppingMall/ShopingMallViewModel.swift)

- 즉각적인 트리거를 수신하기 위한 PublishSubjects(**fetchMoreDatas**) 구독

```swift
let fetchMoreDatas = PublishSubject<Void>()

var cursorCounter = 0 /// offset
private let limit = 20

private var isPaginationRequestStillResume = false /// 페이지네이션 시작 여부

...
private func bind() {
    fetchMoreDatas
	.subscribe { [weak self] _ in
	    guard let self = self else { return }
	    self.populateShoppingProducts(offset: self.cursorCounter)
        }
	.disposed(by: disposeBag)
}
```
	
<br>

```swift
private func populateShoppingProducts(offset: Int) {
    isPaginationRequestStillResume = true /// 페이지네이션 시작 flag

    shoppingAPI.populateShoppingProducts(offset: offset)
        .subscribe { [weak self] products in
            guard let self = self else { return }

            switch products {
            case .success(let products):
		self.handleShoppingProducts(products: products)
                self.isPaginationRequestStillResume = false /// 페이지네이션 끝 flag

            case .failure(let error):
                self.errorMessage.onNext(error)
            }
        }
        .disposed(by: disposeBag)
}
```

<br>
	
- 데이터를 컬렉션뷰에 추가하고 다음 요청에 대한 페이지 정렬

    - 현재 페이지가 0일 때, 새로운 값을 accept
	
    - 페이지네이션을 진행하여 새로운 페이지를 요청할 때, 기존 값에 새로운 값을 accept

	
```swift
private func handleShoppingProducts(products: Products) {
    if cursorCounter == 0 {
        shoppingList.accept(products.products)
    } else {
        let oldDatas = shoppingList.value
	/// 기존 값을 유지하면서 새로운 값을 accept
        shoppingList.accept(oldDatas + products.products)
    }
    cursorCounter += limit /// 다음 row 
}
```

<br>
<br>

[[ShoppingMallViewController]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Presentation/TabBar/ShoppingMall/ShoppingMallViewController.swift)

- **스크롤이 최하단에 도달**
	
	- 현재 스크롤된 위치 > (전체 content 높이 - collectioinview frame 높이)
	
- PublishSubjects(**fetchMoreDatas**)에 이벤트 전달(onNext)

```swift
collectionView.rx.didScroll
    .throttle(.milliseconds(200), scheduler: MainScheduler.instance)
    .subscribe { [weak self] _ in
        guard let self = self else { return }
        let offSetY = self.collectionView.contentOffset.y /// 현재 스크롤된 위치
        let contentHeight = self.collectionView.contentSize.height /// 전체 content 높이

        if offSetY > (contentHeight - self.collectionView.frame.size.height) {
            self.viewModel.fetchMoreDatas.onNext(())
        }
    }
    .disposed(by: disposeBag)
```

<br>
	
	
> issue

**커서 기반 페이지네이션 (Cursor-based Pagination)**

- 기존에 **오프셋 기반 페이지 방식**으로 구현
	
    - '페이지' 단위로 구분하여 요청한다고 생각
	
    - pageCounter += 1 에서 상품이 *0~19* 다음 *1~20*으로 중복을 가져오는 이슈
	
<img src = "https://user-images.githubusercontent.com/93528918/164889248-fa109d4f-16db-4059-8ec9-a90b8d8d529c.png" width="50%" height="50%">

<br>


- **커서 기반 페이지 방식**이기 때문에 limit로 정한 20을 기준으로 pageCounter에 + limit
	
<img src = "https://user-images.githubusercontent.com/93528918/164889255-c285a95c-ae68-47ab-b9ab-aa8f679cf1f8.png" width="50%" height="50%">

<br>
	


## footerView indicator


페이지네이션을 진행하는 동안 CollectionView Footer Section에 indicator 추가

- **configureSupplementaryView**를 사용하기 위해 **RxDataSources** 임포트하여 dataSource 구성

<details>
<summary> dataSource (configureCell, configureSupplementaryView) 코드 </summary>

<br>

```swift
private lazy var dataSource = RxCollectionViewSectionedReloadDataSource<ShoppingMallSection.ShoppingMallSectionModel>(
        configureCell: { dataSource, collectionView, indexPath, item in
            switch item {
            case .firstItem(let product):
                let cell = collectionView.dequeueReusableCell(
                    withReuseIdentifier: ShoppingMallCell.reuseIdentifier,
                    for: indexPath
                ) as! ShoppingMallCell
                cell.configure(product: product)
                return cell
            }
        }, configureSupplementaryView: { [weak self] _, collectionView, kind, indexPath in
            guard let self = self else { return UICollectionReusableView() }
            switch kind {
            case UICollectionView.elementKindSectionFooter:
                let footer = collectionView.dequeueReusableSupplementaryView(
                    ofKind: kind,
                    withReuseIdentifier: ShoppingMallFooterView.reuseIdentifier,
                    for: indexPath
                ) as! ShoppingMallFooterView
                self.setFooterView(footer: footer)
                return footer
            default:
                assert(false, "Unexpected element kind")
            }
        }
    )
```



</div>
</details>

<br>
<br>

[[ShopingMallViewModel]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Presentation/TabBar/ShoppingMall/ShopingMallViewModel.swift)
  
- 즉각적인 트리거를 수신하기 위한 PublishSubjects(**isLoadingSpinnerAvaliable**)
	
- CollectionView의 Footer Section 띄울지 여부 판단하기 위함

```swift
let isLoadingSpinnerAvaliable = PublishSubject<Bool>()

private func populateShoppingProducts(offset: Int, isRefreshControl: Bool) {
	...

        isPaginationRequestStillResume = true
        isLoadingSpinnerAvaliable.onNext(true) /// FooterView -> O

        if cursorCounter == 0 {
            isLoadingSpinnerAvaliable.onNext(false)
        }

        shoppingAPI.populateShoppingProducts(offset: offset)
            .subscribe(onNext: { [weak self] products in
                guard let self = self else { return }
		...
		self.isLoadingSpinnerAvaliable.onNext(false) /// FooterView -> X         
	})
        .disposed(by: disposeBag)
}
```

<br>
<br>

[[ShoppingMallViewController]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Presentation/TabBar/ShoppingMall/ShoppingMallViewController.swift)
  
- Footer Section에 띄울 Indicator 생성

```swift
private let footerView = UIActivityIndicatorView()
...
private func setFooterView(footer: UICollectionReusableView) {
    footer.addSubview(self.footerView)
    self.footerView.frame = CGRect(x: 0, y: 16,
                                   width: collectionView.contentSize.width,
                                   height: 50)
    self.footerView.startAnimating()
}
```

<br>

- isLoadingSpinnerAvaliable을 구독
	
- isLoadingSpinnerAvaliable의 Bool값에 따라 Footer Section Size 조절

```swift
private func binding() {
    viewModel.isLoadingSpinnerAvaliable
        .subscribe { [weak self] isAvailable in
            guard let isAvailable = isAvailable.element,
                  let self = self else { return }
            self.layout.footerReferenceSize = isAvailable || self.viewModel.cursorCounter == 0 ?
            CGSize.zero :
            CGSize(width: self.collectionView.bounds.width, height: 100)
        }
        .disposed(by: disposeBag)
}
```

<br>
	
## Pull Refresh(새로고침)


[[ShopingMallViewModel]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Presentation/TabBar/ShoppingMall/ShopingMallViewModel.swift)
  
- 새로고침 **실행 여부**를 수신하기 위한 PublishSubjects(**refreshControlAction**)
- 새로고침 **완료 여부**를 수신하기 위한 PublishSubjects(**refreshControlCompelted**)

```swift
let refreshControlAction = PublishSubject<Void>()
let refreshControlCompelted = PublishSubject<Void>()

...
private var isRefreshRequstStillResume = false /// 새고고침 flag
```

<br>

```swift
private func bind() {
    fetchMoreDatas
        .subscribe { [weak self] _ in
            guard let self = self else { return }
            self.populateShoppingProducts(offset: self.cursorCounter,
                                          isRefreshControl: false)
        }
        .disposed(by: disposeBag)

    /// 새로고침 제어
    refreshControlAction
        .subscribe { [weak self] _ in
            guard let self = self else { return }
            self.refreshControlTriggered()
        }
        .disposed(by: disposeBag)
}

private func populateShoppingProducts(offset: Int, isRefreshControl: Bool) {
    if isPaginationRequestStillResume || isRefreshRequstStillResume { return }
    self.isRefreshRequstStillResume = isRefreshControl

    ...
    shoppingAPI.populateShoppingProducts(offset: offset)
        .subscribe(onNext: { [weak self] products in
            guard let self = self else { return }

            self.handleShoppingProducts(products: products)
            self.isRefreshRequstStillResume = false /// 새로고침 끝 flag
            self.refreshControlCompelted.onNext(()) /// "완료" 이벤트 전달
	    ...
        })
        .disposed(by: disposeBag)
}
```

<br>

- 새로고침 제어가 트리거되는 즉시, 이전 요청이 취소되고 이전 데이터 삭제

```swift
private func refreshControlTriggered() {
    isPaginationRequestStillResume = false
    cursorCounter = 0
    shoppingList.accept([])
    populateShoppingProducts(offset: cursorCounter,
                             isRefreshControl: true)
}
```

<br>
<br>

[[ShoppingMallViewController]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Presentation/TabBar/ShoppingMall/ShoppingMallViewController.swift)


- 새로고침 이벤트 발생 시, refreshControlAction에 이벤트 전달(onNext)
- ShopingMallViewModel에서 refreshControlCompelted 이벤트를 받으면 새로고침 중단(endRefreshing)

```swift
private let refreshControl = UIRefreshControl()

...
private func binding() {
    refreshControl.rx
        .controlEvent(.valueChanged)
        .bind { [weak self] _ in
            guard let self = self else { return }
            DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
                self.viewModel.refreshControlAction.onNext(())
            }
        }
        .disposed(by: disposeBag)

    viewModel.refreshControlCompelted
        .subscribe { [weak self] _ in
            guard let self = self else { return }
            self.refreshControl.endRefreshing()
        }
        .disposed(by: disposeBag)
}
```

<br>

## 이미지 캐싱

[[UIImageView+Extension]](https://github.com/camosss/MommyTalk_ShoppingMall/blob/main/MommyTalk_ShoppingMall/Extension/UIImageView%2BExtension.swift)
  
- UIImageView를 확장하여 **setImage(with:)** 메서드 생성

- ImageCache의 **retrieveImage(forKey:, options:)** 메서드를 호출
	
    - default로 저장된 키에 캐시가 존재하는 경우, 그 이미지 그대로 사용
	
    - 캐시가 존재하지 않는 경우, 이미지를 다운하고 해당 키에 저장

	
```swift
import Kingfisher

extension UIImageView {
    func setImage(with urlString: String) {
        ImageCache.default.retrieveImage(forKey: urlString, options: nil) { result in
            switch result {
            case .success(let value):
                if let image = value.image { /// 캐시가 존재하는 경우
                    self.image = image
                } else {                     /// 캐시가 존재하지 않는 경우
                    guard let url = URL(string: urlString) else { return }
                    let resource = ImageResource(downloadURL: url, cacheKey: urlString)
                    self.kf.setImage(with: resource)
                }
            case .failure(let error):
                print(error)
            }
        }
    }
}
```

<br>
	
	
	
