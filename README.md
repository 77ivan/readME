<img width="1534" alt="12" src="https://user-images.githubusercontent.com/74236080/143826875-c12c807d-0b03-4c25-8e97-38b79119164d.png">

<br>

# 영자 신문을 앱으로 간편하게, 해외뉴우스<img src = "https://user-images.githubusercontent.com/93528918/149170874-1428e755-5919-4f06-a153-631c55d4e09e.png" width = 80  align = right> 

[프로젝트 세부 내용 보러가기](https://www.notion.so/912caceec73d480982e656c018594c7d)

<a href="https://apps.apple.com/kr/app/%ED%95%B4%EC%99%B8%EB%89%B4%EC%9A%B0%EC%8A%A4/id1596846397
"><img src="https://www.atrinh.com/list/images/download.svg"></a>


<br>

## Skill

- Swift
- MVVM
- Firebase
- Localization
- AutoLayout, StoryBoard, Code Base UI

<br>

## Library

- [Realm]()
- [Alamofire]()
- [SwiftyJSON]()
- [Kingfisher]()
- [SnapKit]()
- [Tabman]()
- [Toast]()
- [SkeletonView]()
- [CHTCollectionViewWaterfallLayout]()
- [IAMPopup]()



<br />

## 기간별 일정
2021.11.16 ~ 21.11.27 (2주)

<br />

| 진행사항 | 진행기간 | 세부사항 |
|:---:| :--- | :--- |
| 아이디어 기획 | 2021.11.16~21.11.17 | 기획서 작성, API 탐색 및 View 구상 |
| UI 개발 | 2021.11.17~21.11.18 | 앱 전체적인 View 개발 |
| 기능 개발 | 2021.11.18~21.11.26 | API 데이터 받아오기 및 전체적인 기능 개발  |
| 앱 심사 제출 및 심사 | 2021.11.26~21.12.03 | 추수감사절 기간이라 지체됨 |
 

<br />


<details>
<summary>출시 프로젝트 세부 기획 항목</summary>

<br />
 
![스크린샷 2021-12-04 오후 4 36 46](https://user-images.githubusercontent.com/74236080/144701789-fa1198e4-0373-4c82-8be7-5921f2074c73.png)

![스크린샷 2021-12-04 오후 4 37 09](https://user-images.githubusercontent.com/74236080/144701790-72d72d18-459f-4568-8603-30263bf6e286.png)
  
</div>
</details>

<br />
<br />
<br />

## View 구성 및 소개

### 해외 뉴스로 세상을 바라보는 시야를 넓히고, 영어 공부까지 !

- 현재 해외에서 인기 급상승중인 주제와 카테고리별 기사들의 목록을 볼 수 있어요.
- 검색을 통해 원하는 주제의 기사를 찾을 수 있어요.
- 마음에 드는 기사를 공유해보세요 !

<br>
<br>

> Trending Topic

	
- **SkeletonView** 라이브러리를 통해 로딩 View 구현
- **WaterfallLayout**을 적용하여 트렌드한 주제의 기사 데이터를 받아와서 Collection View 구성
- Cell을 선택하면 **SlideView**에 각각의 기사 데이터 구성 및 원본 기사 웹뷰로 이동
 
 <br>
	
<img src = "https://user-images.githubusercontent.com/93528918/149971523-e6c13f4b-322e-4835-a459-fd855b06188b.gif" width="30%" height="30%">




<br>
<br>

> Search
 

	
- **SkeletonView** 라이브러리를 통해 검색한 데이터를 받아오는 동안 로딩 View 구현
- Cell을 선택하면 해당 기사 본문 페이지로 이동
 
<br>

<img src = "https://user-images.githubusercontent.com/93528918/149971528-35fd604a-68c0-41d9-ae2c-8385da279827.gif" width="30%" height="30%">



<br>
<br>

> Category


	
- **Tabman, Pageboy** 라이브러리를 통해 카테고리 별 탭페이징 구현
- **하나의 View, Cell, Controller 재사용**
- Section별 하단에 **전체보기 버튼**을 추가하여, Section별로 받아오는 데이터 전체를 표시하는 View로 이동
 
 <br>

<img src = "https://user-images.githubusercontent.com/93528918/149971533-0e9f8dde-712f-49b0-962e-a226f48d359a.gif" width="30%" height="30%">



<br>
<br>

> 기사 본문
 

	
 - **ScrollView**를 적용하여 각 기사의 본문 길이만큼 동적인 높이 조정

 <br>

<img src = "https://user-images.githubusercontent.com/93528918/149971537-27026971-fa90-48e8-99eb-2d8853e17e19.gif" width="30%" height="30%">



<br />
<br />
<br />




## 버전

> [v1.0.1](https://www.notion.so/v1-0-1-2285257857644e7b8916099eb816309a)

- Date값 Format 오류 수정
- 21/12/08 업데이트 완료

<br>

> [v1.0.2](https://www.notion.so/v1-0-2-57a5662ca6c44d94a1c306df9d3b5083)

- Firebase [Analytics, Crashlytics] 적용
- 코드 리펙토링 (API 호출 메서드, Custom View)
- 21/01/10 업데이트 완료


<br>

> [v1.0.3](https://www.notion.so/v1-0-3-a15eb71a45364214adb2073999176486)

- [IAMPopup](https://github.com/camosss/IAMPopup) 라이브러리 적용으로 SlideView 코드 리펙토링
- Category [Entertainment, Sports] - HTTP Headers 이슈
    - HTTP에서 디폴드로 전달되는 헤더가 변경이 되었는데, `accept-language`를 지정해주고 오류 해결
- 21/02/12 업데이트 완료

