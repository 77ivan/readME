<br>


> Github Repository 검색 클라이언트 애플리케이션
> 

<br>

## D**ependence**



- Then, Toast-Swift, SkeletonView, SnapKit
- Kingfisher
- Moya, RxSwift

<br>


## ****Style Guide****



- [StyleShare Swift Style Guide](https://github.com/StyleShare/swift-style-guide)

<br>


## ****Reference****


- [GitHub API](https://developer.github.com/v3/)


<br>
<br>


## 로그인(OAuth)



<br>

> **TODO**

- Github와 App사이의 리디렉션 과정을 통한 Access Token값 받아오기
- 화면 상단 UIBarButtonItem을 통해 로그인 상태 Toggle

<br>

> **ISSUE**

- OAuth Callback 처리
    - Scene Delegate에서 OAuth를 처리하기 위한 Callback이 발생할 때, ViewController에서 로그인 상태가 바로 적용이 되지 않는 이슈
        - Scene Delegate의 window속성을 통해 rootView를 MainTapController로 다시 전환해주면서 해결
        - RxSwift를 사용하여 Callback을 통해 받아오는 Access Token값을 구독하고 이벤트를 감지해서 UI 변경을 구현하는 방식으로 개선해봐야겠다.
    
- Access Token값 보관 (UserDefaults/Keychain 고민)
    - 해당 프로젝트에서는 UserDefaults에서 Access Token값을 관리
    - 하지만 Access Token같은 경우는 민감한 정보이기 때문에 Keychain을 통해 암호화된 데이터로 저장하는 것이 더 적합하다고 판단되어 추후 개선 예정
    
- Property Wrapper를 사용하여 UserDefaults에 Access Token값 저장
    - 휴먼에러를 방지하면서 UserDefaults에 저장된 Access Token을 관리하기 위한 고민
        - Swift5.1에서 추가된 기능인 Property Wrapper를 적용
        - wrappedValue의 getter는 UserDefaults를 통해 object를 가져와서 반환하고, setter는 UserDefaults를 통해 값을 저장하는 방식으로 구현
    - UserDefaults 같은 경우는 key값만 다르고 같은 코드가 반복되기 때문에, 단순히 property에 값을 대입하는 코드만으로 저장이 가능하고, Enum을 통해 관리하기 수월하다고 느낌
    - UserDefaults뿐만 아니라 반복되는 로직들을 property에 연결하여 사용이 가능해 보여, 추후 다른 로직에도 적용해봐야겠다.

<br>


## 검색



<br>

> **TODO**

- SearchBar를 통해 검색한 단어에 해당하는 Repositories 검색
- 로그인된 상태일 때, Star버튼을 눌러 자신의 계정 Star List에 연동하여 추가

<br>

> **ISSUE**

- Pagination
    - scrollViewDidScroll 메서드에서 스크롤을 Bottom에 닿을 때의 제약을 줬을 때, 계속된 호출로 무분별하게 데이터가 추가되는 이슈
    
    - 해결한 방법
        - Enum case로 현재 View상태(isNow, isLoading)를 구분
        - 스크롤이 TableView Bottom에 닿을 때, Pagination 함수 호출
            - 검색 결과의 count가 30개가 넘었을 때만 호출
        - Pagination 함수 내에서 page값 변수를 생성하여 +1을 해준 뒤 View상태 값(isLoading)을 변경
        - API를 호출하여 Repositories 응답 배열에 append 시켜준 뒤, View 상태값(isNow) 다시 변경
    
    - RxSwift를 사용한 Pagination 구현 방식을 찾아보니, Throttle를 통해 시간 간격을 두고 이벤트를 발생시켜 Pagination 구현이 가능해 보여 해당 방식으로 적용해봐야겠다.
    
- Star버튼을 통한 Star/UnStar 기능 구현
    - 검색 결과 Repositories 응답값과 사용자가 Star한 Repositories 응답값 비교를 통한 분기 처리
        - 검색 결과에 로그인한 사용자가 Star했다는 응답값이 없기 때문에 사용자가 Star한 Repositories를 불러와서 두 배열을 비교하는 식으로 접근
        - 검색 response에 `var starred: Bool? = false`를 추가하여 두 배열이 일치하는 값에 true 대입
        - true로 변경한 Repositories를 데이터 소스와 연결하는 과정에서 해결하지 못함
    - 좀 더 다른 방식으로 접근을 해봐야겠다.

<br>

## 프로필



<br>

> **TODO**

- 사용자가 로그인이 되어있지 않은 상태일 때, 로그인 유도
- 로그인된 상태일 때, 사용자 정보 띄우기
- 로그인된 사용자가 Star한 Repositories 목록 띄우기
- Repositories의 Star버튼을 통해 Star/Unstar 가능

<br>

> **ISSUE**

- 2개의 Section(사용자 정보, Starred Repositories)으로 구성된 TableView, RxDataSources 적용 시도
    - 서로 다른 데이터 타입과 서로 다른 Cell로 구성되어 있는 TableView
    - 반응형 프로그래밍에 대한 학습이

<br>

## 프로젝트 회고



- 프로젝트를 시작할 때 API 명세서 작성이 필요함을 느꼈다.
    - 진행 과정에서 중간중간 API 문서를 이해하고 적용하는 데에 꽤 많은 시간이 소요됐다.
    - 처음에 기획/API 명세서를 잘 정리해둔다면 좀 더 시간을 지체하지 않을 것 같다.

- 반응형 프로그래밍과 아키텍쳐 패턴을 이론이 아닌 미니 프로젝트를 통한 학습의 필요성을 느꼈다.
    - 이론으로만 공부한 상태에서 막상 프로젝트에 적용을 하려니 혼란스럽게만 느껴졌다.
    - 적재적소에 잘 사용할 수 있도록 여러 코드를 읽고 구성해보는 학습을 해야겠다.

<br>
<br>
