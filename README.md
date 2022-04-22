

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


[ShoppingTarget]

- Moya

API를 통해 제공할 기능들을 열거형으로 정의한 후, extension으로 확장해서 TargetType 프로토콜을 채택하여 Moya에서 제공하는 기본적인 네트워킹 레이어를 구축 

<br>

[ShoppingAPI]

- 프로토콜 (ShoppingAPIProtocol)

추후에 아키텍처를 **재사용성, 확장성**이 좋게 만들기 위해서 프로토콜 생성

<br>

- Generic 타입을 사용하여 **request 코드** 재사용

에러 핸들링을 위해 request를 **Single**로 wrapping해서 사용, 응답값 또는 에러를 발행 (구독 시, success와 error 두 가지 이벤트 처리)

<br>

- Observable<T> → Single<T> 리펙토링
	
    - Network Result 에러의 경우, 계속해서 요청을 하는 것이 아닌 error 이벤트를 발행하여 dispose 처리가 되어야한다.
	
    - Network Result 자체를 success 이벤트로 발행하여 에러 핸들링과 동시에 구독 관리를 원활하게 할 수 있도록 할 수 있다.

<br>
<br>

[ShopingMallViewModel]

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


- 즉각적인 트리거를 수신하기 위한 PublishSubjects(**fetchMoreDatas**) 구독

```swift
let fetchMoreDatas = PublishSubject<Void>()

private var pageCounter = 0 /// offset(페이지)
private var isPaginationRequestStillResume = false /// 페이지네이션 시작 여부

...
private func bind() {
    fetchMoreDatas
	.subscribe { [weak self] _ in
	    guard let self = self else { return }
	    self.populateShoppingProducts(offset: self.pageCounter)
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
    if pageCounter == 0 {
        shoppingList.accept(products.products)
    } else {
        let oldDatas = shoppingList.value
	/// 기존 값을 유지하면서 새로운 값을 accept
        shoppingList.accept(oldDatas + products.products)
    }
    pageCounter += 1 /// 요청 페이지 +1
}
```

<br>
<br>

[ShoppingMallViewController]

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

## footerView indicator


페이지네이션을 진행하는 동안 CollectionView Footer Section에 indicator 추가

- **configureSupplementaryView**를 사용하기 위해 **RxDataSources** 임포트하여 dataSource 구성

<details>
<summary> dataSource (configureCell, configureSupplementaryView) 코드</summary>
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

<br>
</div>
</details>

<br>
<br>


	
	
	
	
