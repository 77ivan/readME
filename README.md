
<details>
<summary>Phase 1. iOS</summary>
<div markdown="1">    
	111
2222
	3333
</div>
</details>


- 11/16(화)
    - API 탐색 및 테스트
        
        [Rapid API](https://rapidapi.com/microsoft-azure-org-microsoft-cognitive-services/api/bing-news-search1) - [Bing News Search API](https://docs.microsoft.com/en-us/rest/api/cognitiveservices-bingsearch/bing-news-api-v7-reference#news-categories-by-market)
        
        <aside>
        👉 헤더
        
        </aside>
        
        x-rapidapi-host: [bing-news-search1.p.rapidapi.com](http://bing-news-search1.p.rapidapi.com/)
        
        x-rapidapi-key: 7a44d2c014msh6b22325e916294fp1a78f9jsn584601bd034f
        
        1. 인기 급상승
        
        [https://bing-news-search1.p.rapidapi.com/news/trendingtopics?cc=US&textFormat=Raw&safeSearch=Off](https://bing-news-search1.p.rapidapi.com/news/trendingtopics?cc=US&textFormat=Raw&safeSearch=Off)
        
        ```swift
        **title: 기사 제목 (String)
        subTitle: 서브 제목 (String) json["value"][idx]["query"]["text"]
        postImage: 기사 이미지**
        **newsSearchUrl: 관련 기사 url (String)
        webSearchUrl: 해당 키워드 검색 url (String)
        providerOrganization: ["image"]["provider"][0]["name"] 기사 제공자 이름 (String)**
        ```
        
        1. 카테고리
        
        [https://bing-news-search1.p.rapidapi.com/news?category=\(category)&cc=US&safeSearch=Off&textFormat=Raw](https://bing-news-search1.p.rapidapi.com/news?category=Sports_Soccer&cc=US&safeSearch=Off&textFormat=Raw)
        
        - **뉴스**(비즈니스, 정치, products, 건강)
        - **연예**(영화 및 티비, 음악)
        - **스포츠**(축구, NBA(농구), MLB(야구), NFL(내셔널 풋볼리그), 골프, 테니스, NHL(하키))
        - **테크놀로지**(기술, 과학)
        - **US**(Northeast, South, Midwest, West)
        - **World**(아메리카, 유럽, 아시아, 아프리카, 중동)
        
        ![스크린샷 2021-11-16 오후 9.59.08.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1ca45517-45a0-42f8-a462-ca34aee58eb0/스크린샷_2021-11-16_오후_9.59.08.png)
        
        ```swift
        **name: 기사 제목 (String)
        description: 짤린 기사 본문 (String)
        ["image"]["thumbnail"]contentUrl: 기사 이미지 (String)
        webSearchUrl: 해당 키워드 검색 url (String)
        url: 기사 본문 (String)
        datePublished: 발행 날짜 (String)
        providerImage: json["value"][idx]["provider"][0]["image"]["thumbnail"]["contentUrl"]: 기사 제공자 이미지(String)
        providerOrganization: json["value"][idx]["provider"][0]["name"]: 기사 제공자 이름 (String)**
        ```
        
        1. 검색
        
        [https://bing-news-search1.p.rapidapi.com/news/search?q=\(searchText)&textDecorations=true&cc=US&freshness=Day&textFormat=Raw&safeSearch=Off](https://bing-news-search1.p.rapidapi.com/news/search?q=messi&textDecorations=true&cc=US&freshness=Day&textFormat=Raw&safeSearch=Off)
        
        ```swift
        **category: 카테고리 (String)
        name: 기사 제목 (String)
        description: 짤린 기사 본문 (String)
        url: 기사 본문 (String)**
        ["**image**"]["**thumbnail"]contentUrl: 기사 이미지 (String)
        datePublished: 발행 날짜 (String)
        providerImage: json["value"][idx]["provider"][0]["image"]["thumbnail"]["contentUrl"]: 기사 제공자 이미지(String)
        providerOrganization: json["value"][idx]["provider"][0]["name"]: 기사 제공자 이름 (String)**
        ```
        
    - View, 기능 구상
        
        <aside>
        ✅ 뷰
        
        </aside>
        
        - 인기 급상승
        - 검색
        - 카테고리별
        - 스크랩 목록
        
        <aside>
        ✅ 기능
        
        </aside>
        
        - Alamofire, SwiftyJSON → API 데이터 가져오기
        - KingFisher → 기사 이미지
        - `카테고리` 별 필터
        - WebView → 기사 본문을 웹으로 전환
        - JGProgressHUD → 기사 로드되는 동안 로딩
        - Pagenation
        - 기사 스크랩 → Realm?, 스크랩하면 확인 Toast_Swift 띄우기
        - NWPathMonitor → 네트워크 고려
        - UIActivityViewController → 기사 공유
        

---

- 11/17(수)
    - 스토리보드로 전체적인 View 만들기, 화면 전환
        - WaterfallLayout
        
        ![simulator_screenshot_8C1DFC17-BE23-4610-900E-83C86EE87F39.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d4cc316d-1b1b-492e-863d-1a214849d50b/simulator_screenshot_8C1DFC17-BE23-4610-900E-83C86EE87F39.png)
        
        - ScrollView
        
        ![simulator_screenshot_7AB6E57A-E014-4F05-822E-3B6F130AC4A7.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e565bdb5-2adc-4391-a99b-254cf938fa9d/simulator_screenshot_7AB6E57A-E014-4F05-822E-3B6F130AC4A7.png)
        
        - SlideView
        
        ![simulator_screenshot_B00AE471-224E-4238-BC62-1B0B5022C625.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1f079a9-a8b6-4959-bf52-abf371667760/simulator_screenshot_B00AE471-224E-4238-BC62-1B0B5022C625.png)
        
        카테고리 View는 좀 더 고민,,
        

---

- 11/18(목)
    - 카테고리 View 구현
        
        [Tabman 라이브러리](https://github.com/uias/Tabman)
        
        ![simulator_screenshot_80BCE020-EC64-42F2-B67F-9FCFE7CC8DDE.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e854b558-bde6-4e04-a98a-a047d2ea449e/simulator_screenshot_80BCE020-EC64-42F2-B67F-9FCFE7CC8DDE.png)
        
        - 더보기 버튼 추가
        
        각 카테고리별 메인화면에 3개씩만 기사를 띄우고, 나머지 기사는 더보기 버튼을 만들어서 화면을 전환하려한다.
        
        그런데, Footer View에 버튼을 생성하는 과정에서 아래와 같이 `TableView` 를 스크롤 할 때 `TableViewFooterView` 가 내려가는 것이 아니라 Section이 끝날 때까지 고정되어 있게 된다.
        
        이 문제는 TableView의 style을 `plain` → `grouped` 으로 바꿔주면서 해결했다.
        
        ![Simulator Screen Shot - iPhone 13 Pro - 2021-11-18 at 19.29.51.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2308a7e5-bd42-49ae-9bb5-90fbd8597c69/Simulator_Screen_Shot_-_iPhone_13_Pro_-_2021-11-18_at_19.29.51.png)
        
        해결 !
        
        ![Simulator Screen Shot - iPhone 13 Pro - 2021-11-18 at 19.32.44.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6e88b35a-c1e0-4bc6-adf5-0589611ff9c3/Simulator_Screen_Shot_-_iPhone_13_Pro_-_2021-11-18_at_19.32.44.png)
        
    - API 데이터 .GET
        
        카테고리는 섹션별로 어떻게 받아오는지 해결해야함.
        
        ![simulator_screenshot_AE0B80BA-3DCC-4400-88AD-2FFEB6772516.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2dfd528f-adaf-480f-88a7-bc5de2a9aa15/simulator_screenshot_AE0B80BA-3DCC-4400-88AD-2FFEB6772516.png)
        
        ![simulator_screenshot_4E5B584B-5E3F-4272-9CB1-AF183871C3CE.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3294c368-ff91-4223-8c76-dd1227852e31/simulator_screenshot_4E5B584B-5E3F-4272-9CB1-AF183871C3CE.png)
        

---

- 11/19(금)
    - 검색기능 구현
        
        검색기능이 없을 때 화면 구현
        
        ![simulator_screenshot_F3062224-E274-45E5-9B76-78C6EFBE7DC0.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd4bd9a7-bf1a-4d80-bc88-92e69690e1fa/simulator_screenshot_F3062224-E274-45E5-9B76-78C6EFBE7DC0.png)
        
    - 카테고리 API 데이터 넣기
        - 6개의 카테고리별로 뷰컨을 생성하다보니 같은코드가 6개가 반복되는거 리펙토링 생각
        - 이미지 뷰의 데이터가 없으면 뷰에서 이미지 뷰를 삭제할 수 있는지

---

- 11/21(일)
    - 기사 본문으로 데이터 전달
        - 검색, 카테고리 데이터
    - 공유, 네트워크
        - 기사 원본 url 공유
    - 기사 url 웹뷰로 전환
        - UILabel에 링크거는 것과 underline 표시하는법 배움
    - Trending, Search, Category별로 API
        - Trending
        
        ![Simulator Screen Shot - iPhone 13 Pro - 2021-11-21 at 21.33.57.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/38c5eabd-67c4-4204-bb50-fd10792ce7ff/Simulator_Screen_Shot_-_iPhone_13_Pro_-_2021-11-21_at_21.33.57.png)
        
        ![Simulator Screen Shot - iPhone 13 Pro - 2021-11-21 at 21.34.00.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5e95390e-ca3c-4afb-8eec-1e84ff206cbb/Simulator_Screen_Shot_-_iPhone_13_Pro_-_2021-11-21_at_21.34.00.png)
        
        - 검색
        
        ![simulator_screenshot_58205D2D-8A9C-489A-AB6E-947C8CD6B990.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1138a182-de0f-4920-8bb8-34ab3b410eb1/simulator_screenshot_58205D2D-8A9C-489A-AB6E-947C8CD6B990.png)
        

---

- 11/22(월)
    - 카테고리 뷰컨 재사용
        1. enum으로 각 페이지별 케이스를 정한다.
        
        ![스크린샷 2021-11-22 오후 7.23.28.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d7bceb7-d413-42de-83ab-4a583a85b9c1/스크린샷_2021-11-22_오후_7.23.28.png)
        
        1. 인덱스에 접근해서 케이스별 데이터를 넘겨준다.
        
        ```swift
        func viewController(for pageboyViewController: PageboyViewController, at index: PageboyViewController.PageIndex) -> UIViewController? {
            let vc = viewControllers[index] as? CategorySectionViewController
            vc?.sectionName = Category.allCases[index].description
            return vc
        }
        ```
        
        ❗️ Article 데이터를 받는 프로퍼티를 하나만 생성하고 sectionURL 인덱스별로 데이터를 담아야한다.
        
        → Article에 섹션이름 프로퍼티를 하나 추가해서 필터처리해줬음 (잭님 피드백)
        
    - 타이틀 폰트 적용
        
        ![스크린샷 2021-11-23 오전 1.29.24.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9ea9eba7-6af1-43c0-9104-c3e3b97c8611/스크린샷_2021-11-23_오전_1.29.24.png)
        
        ![스크린샷 2021-11-23 오전 1.29.32.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0c3dd6c9-3b2e-497c-9a45-dfc0db0d6311/스크린샷_2021-11-23_오전_1.29.32.png)
        
        ![스크린샷 2021-11-23 오전 1.29.38.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6c180f04-b0e9-46d5-b492-d23033cf888e/스크린샷_2021-11-23_오전_1.29.38.png)
        

---

- 11/23(화)
    - 카테고리별로 받은데이터 "전체보기" 컨트롤러로 데이터 전달
        
        tag값으로 해당 섹션의 데이터를 필터처리해서 seeMore페이지로 전달
        
        ```swift
        CategorySectionViewController
        let sectionData = article.filter{$0.sectionName == sectionURL[button.tag]}
        vc.**article** = sectionData
        
        SeeMorePageViewController
        var **article**: [Article] = []
        ```
        
    - 스크롤뷰 제약 다시 설정
        
        
    - api콜한거 realm데이터에 저장
        - `Realm` 에 데이터를 List로 저장
        
        **날짜**, **섹션이름**은 처음 값을 호출로 가져올 때, 비교하기 위해
        
        ![스크린샷 2021-11-23 오후 11.24.25.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e04d899-1edf-4d3b-9f88-12985318dfff/스크린샷_2021-11-23_오후_11.24.25.png)
        
        ![,.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ebd20315-53f6-4e14-b007-99386630c750/.png)
        
        - Realm 데이터에 오늘 저장된 날짜가 없고 AND 저장된 섹션이 없다면 api콜로 데이터를 가져와서 Realm에 저장
        
        ```swift
        if localRealm.objects(SaveArticle.self)
        	.filter("saveDate == '\(todayDateString)' AND sectionName == '\(urlString)'")
        	.isEmpty {
        
        } else {
        		***저장된 데이터 로드***
        }
        ```
        
        ❗️ SaveArticle 에서 List 값인 articleModels에 접근하는 코드
        
        ```swift
        tasks?.filter("sectionName == %@", sectionURL[indexPath.section]).first?.articleModels[indexPath.row]
        ```
        
        <aside>
        ❓ 전날 Realm에 저장된 데이터는 삭제해줘야하는지
        
        </aside>
        

---

- 11/24(수)
    - API 탐색
        
        
    - 스켈레톤 구현
        
        1. api를 가져오는 동안 스켈레톤을 띄워야하는부분을 헷갈렸음
        2. TableView는 쉽게 적용했는데, CollectionView는 다른방법이 있는건지 적용이 안됐음(스켈레톤 갯수도 설정해줘야했음)
        
        ![simulator_screenshot_E571B6A4-2B36-4585-BCF7-F9EC34D3EDC1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/318a5e9b-1b84-4093-baa4-a987eb539bca/simulator_screenshot_E571B6A4-2B36-4585-BCF7-F9EC34D3EDC1.png)
        
        ![simulator_screenshot_B138DDB6-CF34-4827-A767-DEFB1F796152.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9314455c-aeda-45af-9ea1-09fc158ab5cb/simulator_screenshot_B138DDB6-CF34-4827-A767-DEFB1F796152.png)
        

---

- 11/25(목)
    - 트렌딩 로딩뷰
        - numberOfItemsInSection - collectionView와 skeletonView의 갯수를 동일하게 설정해줘야한다.
        
        ![simulator_screenshot_79F25290-F98D-4098-A69F-B63AE2E21429.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2badd672-6535-4da6-8a63-705492c4497f/simulator_screenshot_79F25290-F98D-4098-A69F-B63AE2E21429.png)
        
    - 다크모드 고려
        
        
    - 설정
        - Notice → view
        - Send feedback → 메일로 연결
        - Leave App Store Ratings and Reviews → 앱스토어로 연결
        - Open Source License → tableView
    - 기사본문 날짜 String 값 포맷
        
        
    - 앱 아이콘, 앱 이름 다국어지원
        
        ![스크린샷 2021-11-26 오전 12.08.04.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2b655154-36dc-43b3-a9e1-cac6f4c2b349/스크린샷_2021-11-26_오전_12.08.04.png)
        
        ![스크린샷 2021-11-26 오전 12.08.51.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/31e28af3-347e-42ba-8e56-4ebfe203838b/스크린샷_2021-11-26_오전_12.08.51.png)
        

---

- 11/26(금)
    - 트렌딩 로딩뷰 수정
        - skeleton 데이터 소스에서 갯수를 임의로 정해주면 되는거였음
    - 앱스토어 스크린샷 만들기
        
        ![스크린샷 2021-11-26 오후 7.40.12.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/866d59ea-557f-4a31-9c29-999fb6c2d3ec/스크린샷_2021-11-26_오후_7.40.12.png)
        
    - 공지사항 적기
        - 간단한 앱소개
        - 매일 자정에 기사 업데이트 되는거 알리기
        - 카테고리 api 이미지 저화질인거 양해
        - g메일 등록하고 피드백보내는법 알려주기

---

- 11/27(토)
    - 앱스토어 제출
        
        ![스크린샷 2021-11-27 오후 5.21.25.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/48c9df98-d6fe-4215-bb8d-7030edfb67fc/스크린샷_2021-11-27_오후_5.21.25.png)
