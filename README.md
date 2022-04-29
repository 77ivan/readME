## 네이버 영화 API를 활용한 영화 검색

> **RxSwift**와 **MVVM** 패턴을 사용한 영화 검색 클라이언트 애플리케이션
> 

<br>

- 최소 버전: iOS 13
- 다크모드 대응
- 이미지 캐싱

<br>

## TODO

- [Rest API 핸들링](#Rest-API-핸들링)
- [페이지네이션](#페이지네이션)
- [footerView indicator](#footerView-indicator)
- [검색 기능](#검색-기능)
- [로딩 indicator](#로딩-indicator)
- [즐겨찾기 기능](#즐겨찾기-기능)
- [즐겨찾기 새로고침](#즐겨찾기-새로고침)

<br>

## Library

- [Kingfisher](https://github.com/onevcat/Kingfisher)
- [Moya](https://github.com/Moya/Moya)
- [ProgressHUD](https://github.com/relatedcode/ProgressHUD)
- [Realm](https://github.com/realm/realm-swift)
- [RxSwift, RxCocoa](https://github.com/ReactiveX/RxSwift)
- [SnapKit](https://github.com/SnapKit/SnapKit)
- [Then](https://github.com/devxoul/Then)
- [Toast](https://github.com/scalessec/Toast-Swift)

<br>

## Rest API 핸들링

[[SearchMovieTarget]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Network/SearchMovie/SearchMovieTarget.swift)

**Moya**

- API를 통해 제공할 기능들을 열거형으로 정의한 후, extension으로 확장해서 TargetType 프로토콜을 채택하여 Moya에서 제공하는 기본적인 네트워킹 레이어를 구축

<br>

[[SearchMovieAPI]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Network/SearchMovie/SearchMovieAPI.swift)

**프로토콜 (SearchMovieAPIProtocol)**

- 추후에 아키텍처를 **재사용성, 확장성**이 좋게 만들기 위해서 프로토콜 생성

<br>

**Observable → Single 리펙토링**

- Network Result 에러의 경우, **계속해서 요청을 하는 것이 아닌 error 이벤트를 발행하여 dispose 처리가 되어야한다.**

- Network Result 자체를 success 이벤트로 발행하여 에러 핸들링과 동시에 구독 관리를 원활하게 할 수 있도록 할 수 있다.

<br>

[[SearchMovieViewModel]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewModel.swift)

영화 목록

- **BehaviorRelay**: 기존 값을 유지하면서 새로운 값을 accept

- **Relay** : completed, error를 발생하지 않고 Dispose되기 전까지 계속 작동하기 때문에 UI Event에서 사용

```swift
var movieList = BehaviorRelay<[Movie]>(value: [])

private let searchMovieAPI: SearchMovieAPIProtocol
private let disposeBag = DisposeBag()

init(searchMovieAPI: SearchMovieAPIProtocol = SearchMovieAPI()) {
    self.searchMovieAPI = searchMovieAPI
    bind()
}

private func bind() {
    searchMovieAPI
        .populateMovieList(query: query, start: cursor)
        .subscribe { [weak self] movies in
            self.movieList.accept(movies.items) /// 새로운 값을 accept
        })
        .disposed(by: disposeBag)
}
```

<br>

## 페이지네이션

Infinite Scroll 구현

<br>

[[SearchMovieViewModel]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewModel.swift)

- 즉각적인 트리거를 수신하기 위한 PublishSubjects(**fetchMoreDatas**) 구독

- **fetchMoreDatas**에 해당 이벤트를 전달받으면 API 요청 메서드 호출

```swift
let fetchMoreDatas = PublishSubject<Void>()

var startCounter = ParameterValue.start.rawValue /// start(parameter) = 1
private var totalValue = ParameterValue.start.rawValue /// 전체 결괏값
private let limit = ParameterValue.display.rawValue /// display(parameter) = 20

private var isPaginationRequestStillResume = false /// 페이지네이션 진행 여부

...
private func bind() {
    fetchMoreDatas
        .subscribe { [weak self] _ in
            guard let self = self else { return }
            self.populateMovieList(cursor: self.startCounter)
        }
        .disposed(by: disposeBag)
}
```

<br>

> populateMovieList(cursor:)

- **isPaginationRequestStillResume** (페이지네이션 flag)를 기준으로 새로운 데이터 추가
    
    - startCounter(페이지네이션을 통해 값이 limit만큼 더해진)가 검색한 데이터의 전체값보다 크면 마지막 페이지이므로 false
   
    - startCounter가 파라미터에 지정해준 값(start = 1)과 같으면 처음 페이지가 로드된 것이기 때문에 false

```swift
private func populateMovieList(cursor: Int) {
    isPaginationRequestStillResume = true /// 페이지네이션 시작 flag

    if startCounter > totalValue { /// 마지막 페이지
        isPaginationRequestStillResume = false
        errorMessage.onNext(NetworkError.last_page)
        return
    }

    if startCounter == ParameterValue.start.rawValue {
        isPaginationRequestStillResume = false
    }

    searchMovieAPI
        .populateMovieList(query: query, start: cursor)
        .subscribe { [weak self] movies in
            guard let self = self else { return }
            switch movies {
            case .success(let movies):
                self.handleStartCounter(movies: movies)
                self.isPaginationRequestStillResume = false /// 페이지네이션 종료 flag

            case .failure(let error):
                self.errorMessage.onNext(error)
            }
        }
        .disposed(by: disposeBag)
 }
```

<br>

> handleStartCounter(movies:)
 
- 데이터를 TableView에 추가하고 다음 요청에 대한 페이지 정렬
    
    - 현재 페이지가 1(start)일 때, **새로운 값을 accept**
    
    - 페이지네이션을 진행하여 새로운 페이지를 요청할 때, **기존 값에 새로운 값을 accept**

- **커서 기반 페이지네이션** (Cursor-based Pagination)으로 limit으로 정한 display 파라미터의 값만큼 startCounter에 더해줘서 새로운 데이터 요청
    
    - 클라이언트가 가져간 마지막 row의 순서상 다음 row들을 n(display)개 요청

```swift
private func handleStartCounter(movies: Movies) {
    totalValue = movies.total

    if startCounter == ParameterValue.start.rawValue {
        movieList.accept(movies.items)
    } else {
        let oldDatas = movieList.value
        /// 기존 값을 유지하면서 새로운 값을 accept
        movieList.accept(oldDatas + movies.items)
    }
    startCounter += limit /// start += display(20)
}
```

<br>

[[SearchMovieViewController]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewController.swift)

- **스크롤이 최하단에 도달**
    
    - 현재 스크롤된 위치 > (전체 content 높이 - tableView frame 높이 - 100)

- PublishSubjects(**fetchMoreDatas**)에 이벤트 전달(onNext)
    
    - 추후 검색 기능에서 검색 시, fetchMoreDatas 이벤트 중복 이슈가 있기 때문에 startCounter가 파라미터에 지정해준 값(start = 1)보다 클 때, 페이지네이션 이벤트 전달할 수 있도록 조건문 삽입

```swift
tableView.rx
    .didScroll
    .throttle(.milliseconds(200), scheduler: MainScheduler.instance)
    .subscribe { [weak self] _ in
        guard let self = self else { return }
        let offSetY = self.tableView.contentOffset.y /// 현재 스크롤된 위치
        let contentHeight = self.tableView.contentSize.height /// 전체 content 높이

        if offSetY > (contentHeight - self.tableView.frame.size.height - 100) {
            if self.viewModel.startCounter > 1 {
                self.viewModel.fetchMoreDatas.onNext(())
            }
        }
    }
    .disposed(by: disposeBag)
```

<br>

## footerView indicator

페이지네이션을 진행하는 동안 TableView FooterView에 indicator 추가

<br>

[[SearchMovieViewModel]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewModel.swift)

- 이전 **isPaginationRequestStillResume** (페이지네이션 flag)를 PublishSubject(**isLoadingSpinnerAvaliable**)로 대체

- TableView의 FooterView를 띄울지 여부 판단

```swift
let isLoadingSpinnerAvaliable = PublishSubject<Bool>() /// 페이지네이션, footerView indicator
```

<br>

[[SearchMovieViewController]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewController.swift)


- FooterView에 띄울 Indicator 생성

- **isLoadingSpinnerAvaliable**의 Bool값에 따라 FooterView Size 결정

```swift
private lazy var viewSpinner = UIView(
    frame: CGRect(x: 0, y: 0, width: view.frame.size.width, height: 100)
).then {
    let spinner = UIActivityIndicatorView()
    spinner.center = $0.center
    $0.addSubview(spinner)
    spinner.startAnimating()
}

...
private func bind() {
    viewModel.isLoadingSpinnerAvaliable
        .subscribe { [weak self] isAvailable in
            guard let isAvailable = isAvailable.element,
                  let self = self else { return }
            self.tableView.tableFooterView = isAvailable ? self.viewSpinner : UIView(frame: .zero)
        }
        .disposed(by: disposeBag)
    ...
}
```

<br>

## 검색 기능

[[SearchBar]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Custom/SearchBar.swift) - 커스텀 SearchBar 

- 검색 버튼이 눌려진 후 SearchBar의 text를 상위 뷰로 전달하는 **shouldLoadResult** 이벤트 시퀀스

- 검색 버튼을 눌렀을 때 발생하는 이벤트를 구독할 PublishRelay(**searchButtonTapped**)
    
    - **SearchButtonTapped**는 UI에서 발생하는 이벤트이기 때문에 PublishSubject보다 error을 방출하지 않는 **PublishReplay**를 사용하는 것이 더 적합

```swift
var shouldLoadResult = Observable<String>.of("") /// 검색 text 전달
let searchButtonTapped = PublishRelay<Void>()
```

<br>

> bind

- 키보드의 검색 버튼을 눌렀을 때, **searchButtonTapped**이 이벤트가 발생한 걸 알 수 있도록 bind

- **searchButtonTapped** 이벤트가 발생하면 키보드가 내려가는 이벤트 발생

- **searchButtonTapped** 실행 → 검색 text를 전달하는 shouldLoadResult 실행 (SearchButtonTapped가 ShouldLoadResult의 트리거)

```swift
private func bind() {
    self.rx.searchButtonClicked.asObservable()
        .bind(to: searchButtonTapped)
        .disposed(by: disposeBag)

    searchButtonTapped
        .asSignal() /// PublishRelay -> observable()
        .emit(to: self.rx.endEditing) /// 키보드 내려가는 이벤트 방출
        .disposed(by: disposeBag)

		/// SearchButtonTapped가 ShouldLoadResult의 트리거
    self.shouldLoadResult = searchButtonTapped
        .withLatestFrom(self.rx.text) { $1 ?? "" } /// 최신 text를 전달
        .filter { !$0.isEmpty } // 비어있지 않은 경우만
        .debounce(.milliseconds(500), scheduler: MainScheduler.instance)
}
```

<br>

[[SearchMovieViewController]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewController.swift)

- 커스텀 SearchBar의 **searchButtonTapped**을 통해 **shouldLoadResult**가 최신 text(query)를 방출

- 구독 이후의 이벤트만 제공하고, 메인스트림에서 실행하며 error를 방출하지 않아 UI이벤트를 다루는데에 적합하기에 asSignal()을 사용하여 **emit()** 연산자를 통해 이벤트 방출

```swift
private let searchBar = SearchBar()

...
searchBar.shouldLoadResult
    .asSignal(onErrorJustReturn: "")
    .emit(onNext: { [weak self] query in
        guard let self = self else { return }
        self.viewModel.searchResultTriggered(query: query)
    })
    .disposed(by: disposeBag)
```

<br>

[[SearchMovieViewModel]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewModel.swift)

- startCounter를 1로 맞추고 movieList를 빈 배열로 accept

- **fetchMoreDatas**에 이벤트 전달하여 API 요청

```swift
var query = "" /// 검색 text

...
func searchResultTriggered(query: String) {
    self.query = query
    self.startCounter = ParameterValue.start.rawValue
    self.movieList.accept([])
    self.fetchMoreDatas.onNext(())
}
```

<br>

## 로딩 indicator

API 요청을 통해 데이터를 받아오는 동안 로딩 indicator 구현

<br>

[[SearchMovieViewModel]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewModel.swift)

- 로딩 실행 여부를 수신하기 위한 PublishSubject(**isLoadingAvaliable**)

- **isLoadingRequstStillResume**는 로딩 indicator와 emptyView를 구분하기 위한 flag
    
    - 검색 결괏값 0일 때, EmptyView를 로드하는데, 로딩 indicator를 띄우는 동안 emptyView는 사라지게 하기 위함
    
<details>
<summary> 코드 </summary>

<br>
	
```swift
/// SearchMovieViewController
    
viewModel.movieList
    .map { return $0.count <= 0 && !self.viewModel.isLoadingRequstStillResume }
    .bind(to: tableView.rx.isEmpty(
        title: "영화를 검색해보세요!",
        imageName: "film")
    )
    .disposed(by: disposeBag)
```
				  
<br>
	
</div>
</details>			  
    

```swift
let isLoadingAvaliable = PublishSubject<Bool>() /// 검색, indicator
var isLoadingRequstStillResume = false /// 로딩 indicator와 emptyView를 구분하기 위한 flag
```

<br>

- 검색 결과를 트리거하는 메서드에 **true** 이벤트를 전달

```swift
func searchResultTriggered(query: String) {
    self.isLoadingAvaliable.onNext(true)
    self.isLoadingRequstStillResume = true
    ...
}
```

<br>

- API 요청을 통해 검색 결과를 받아왔을 때, **false** 이벤트 전달

```swift
searchMovieAPI
    .populateMovieList(query: query, start: cursor)
    .subscribe { [weak self] movies in
        guard let self = self else { return }
        switch movies {
        case .success(let movies):
            self.handleStartCounter(movies: movies)
            self.isLoadingAvaliable.onNext(false)
            self.isLoadingRequstStillResume = false
            self.isLoadingSpinnerAvaliable.onNext(false)
        case .failure(let error):
            self.errorMessage.onNext(error)
            self.isLoadingAvaliable.onNext(false)
            self.isLoadingRequstStillResume = false
        }
    }
    .disposed(by: disposeBag)
```

<br>

[[SearchMovieViewController]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/SearchMovie/SearchMovieViewController.swift)

- **isLoadingAvaliable**을 구독하여 isAvailable(Bool)값에 따라 indicator start/stop Animating

```swift
viewModel.isLoadingAvaliable
    .subscribe { [weak self] isAvailable in
        guard let isAvailable = isAvailable.element,
              let self = self else { return }
        if isAvailable {
            self.indicator.startAnimating()
        } else {
            self.indicator.stopAnimating()
        }
    }
    .disposed(by: disposeBag)
```

<br>

## 즐겨찾기 기능

**Realm**을 사용하여 즐겨찾기 목록 구현

[[MovieCell]](https://github.com/camosss/MovieProject/blob/main/MovieProject/View/MovieCell.swift) 

[[DetailHeaderView]](https://github.com/camosss/MovieProject/blob/main/MovieProject/View/DetailHeaderView.swift)

- StarButton의 isSelected(Bool)값 이벤트를 수신하기 위한 PublishSubject(**isStarred**)

- 해당 Bool값에 따라 Cell의 StarButton 이미지 변경 (star.fill (true) / star (false))

```swift
private let isStarred = PublishSubject<Bool>()

...
isStarred
    .asSignal(onErrorJustReturn: false)
    .emit(to: starButton.rx.isSelected)
    .disposed(by: disposeBag)
```

<br>

- starButton의 tap 이벤트를 isSelected(Bool)값으로 매핑
- isSelected(Bool)값에 따른 스타버튼 로직 처리 (Star/UnStar)

```swift
starButton.rx.tap
    .asSignal()
    .throttle(.seconds(1))
    .map { [weak self] _ -> Bool in
        guard let self = self else { return false }
        return self.starButton.isSelected
    }
    .distinctUntilChanged()
    .emit(onNext: { [weak self] isSelected in
        guard let self = self else { return }
        if isSelected {
            self.requestUnStar()
        } else {
             self.requestStar()
         }
    })
    .disposed(by: disposeBag)
```

<br>

> **Star 로직**
> 

[[Movie]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Network/SearchMovie/Model/Movie.swift)

- Realm에 저장할 데이터 객체(Movie) 재정의

- 데이터 객체에 따로 ID값이 없기 때문에 **link**를 **primaryKey**로 등록
    
    - **code**값으로 식별 ()
    
    https://movie.naver.com/movie/bi/mi/basic.nhn?code=187347
    
    https://movie.naver.com/movie/bi/mi/basic.nhn?code=134898
    

```swift
class Movies: Codable {
    var lastBuildDate: String
    var total, start, display: Int
    var items = List<Movie>()

    var itemArray: [Movie] {
        get {
            return items.map{$0}
        }
        set {
            items.removeAll()
            items.append(objectsIn: newValue)
        }
    }
}

class Movie: Object, Codable {
    @Persisted var title: String?
    @Persisted var link: String?
    @Persisted var image: String?
    @Persisted var subtitle: String?
    @Persisted var pubDate: String?
    @Persisted var director: String?
    @Persisted var actor: String?
    @Persisted var userRating: String?

    override class func primaryKey() -> String? {
        return "link"
    }
}
```

<br>

[MovieCell](https://github.com/camosss/MovieProject/blob/main/MovieProject/View/MovieCell.swift)

- Realm에 저장된 데이터를 불러와서 **movie.link*** 값을 비교(filter)하여 즐겨찾기 목록에 추가 및 삭제

<br>

> issue
> 

StarButton을 반복적으로 눌러서 저장, 삭제를 반복하면 해당 에러 발생.

- **'Object has been deleted or invalidated.’**

JSON 데이터를 Realm Object로 변환하면 DB에 연결되지 않은 인스턴스가 생성되는데, 그 객체를 같은 객체로 로컬 데이터베이스에서 추적한다.

그래서 Movie를 즐겨찾기에 추가하고난 뒤, 삭제를 하게되면 해당 인스턴스는 참조가 사라져 **Invalid Object**가 되서 다시 한번 추가하려고 하면 해당 에러가 발생하는 것.

<br>

- 해결책
    
    - Realm 객체에 Json을 전달하는 대신 **객체를 먼저 복사한 후 복제된 객체를 저장한다.**
    
    - 그러면 목록에 추가할 때마다 새로운 인스턴스가 생성되어 삭제가 되서 **Invalid Object**가 되더라도 또 다시 새로 복제된 객체가 목록에 추가되는 로직

```swift
extension MovieCell {
    private func requestStar() {
        self.isStarred.onNext(true)

        /// add realm
        if realm.objects(Movie.self)
            .filter("link == '\(self.movie?.link ?? "")'").isEmpty {
            try! realm.write({
                /// realm 객체에 Json을 전달하는 대신 객체를 먼저 복사한 후 복제된 객체를 저장
                let copyMovie = self.realm.create(Movie.self, value: movie!, update: .all)
                self.realm.add(copyMovie)
                ProgressHUD.show(StarStatus.star.description, icon: .star)
            })
        }
    }

    private func requestUnStar() {
        self.isStarred.onNext(false)

        /// remove realm
        if !realm.objects(Movie.self)
            .filter("link == '\(self.movie?.link ?? "")'").isEmpty {
            try! realm.write({
                let copyMovie = self.realm.create(Movie.self, value: movie!, update: .all)
                self.realm.delete(copyMovie)
                ProgressHUD.show(StarStatus.unstar.description, icon: .star)
            })
        }
    }
}
```

<br>

## 즐겨찾기 새로고침

[[FavoritesViewModel]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/Favorites/FavoritesViewModel.swift)

- 새로고침 **실행 여부**를 수신하기 위한 PublishSubjects(**refreshControlAction**)

- 새로고침 **완료 여부**를 수신하기 위한 PublishSubjects(**refreshControlCompelted**)

```swift
let refreshControlAction = PublishSubject<Void>() /// 새로고침 실행 여부를 수신
let refreshControlCompelted = PublishSubject<Void>() /// 새로고침 완료 여부를 수신
```

<br>

- refreshControlAction을 구독하여 이벤트 수신

- refreshControlTriggered() → 새로고침 제어가 트리거되는 즉시, 데이터를 다시 불러온 뒤 완료 이벤트(refreshControlCompelted) 전달

```swift
private func bind() {
    favoriteList.accept(favorites.map{$0})

    refreshControlAction
        .subscribe { [weak self] _ in
            guard let self = self else { return }
            self.refreshControlTriggered()
        }
        .disposed(by: disposeBag)
}

private func refreshControlTriggered() {
    favoriteList.accept(favorites.map{$0})
    refreshControlCompelted.onNext(()) /// 완료여부 이벤트 전달
}
```

<br>

[[FavoritesViewController]](https://github.com/camosss/MovieProject/blob/main/MovieProject/Presentation/Favorites/FavoritesViewController.swift)

- 새로고침 이벤트 발생 시, refreshControlAction에 이벤트 전달(onNext)

- ShopingMallViewModel에서 refreshControlCompelted 이벤트를 받으면 새로고침 중단(endRefreshing)

```swift
refreshControl.rx
     .controlEvent(.valueChanged)
     .bind { [weak self] _ in
         guard let self = self else { return }
         DispatchQueue.main.asyncAfter(deadline: .now() + 0.5) {
            self.viewModel.refreshControlAction.onNext(()) /// 새로고침 실행여부 이벤트 전달
         }
     }
     .disposed(by: disposeBag)

viewModel.refreshControlCompelted
	    .subscribe { [weak self] _ in
         guard let self = self else { return }
         self.refreshControl.endRefreshing()
     }
     .disposed(by: disposeBag)
```

<br>

## 아쉬운 점

화면간의 StarButton Image 동기화 (검색 화면, 영화 상세 화면, 즐겨찾기 화면)

- 영화 검색 후, 상세 화면이나 즐겨찾기 화면으로 push하여 스타버튼을 클릭해 Realm에 해당 데이터 객체를 저장한 뒤, 다시 검색 화면으로 돌아왔을 때 StarButton 이미지 동기화 처리에 대한 아쉬움
- 화면 전환 시, 검색 화면을 채우는 moveList를 빈 배열로 accept하여 다시 검색 View로 pop했을 때 emptyView를 로드하는 것으로 대체

<br>

**'Object has been deleted or invalidated.’** 에러 대응 (즐겨찾기)

- 검색 View에서는 복제된 객체를 생성하여 반복적으로 저장 및 삭제가 가능하나, 같은 Cell을 사용하는 즐겨찾기 View에서는 해당 에러 발생


