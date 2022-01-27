
# 서비스 레벨 프로젝트


### 서비스 레벨의 API/기획 명세와 디자인 리소스를 바탕으로 iOS 앱개발

<br>

![Badge](https://img.shields.io/badge/Xcode-13.0-blue) 
![Badge](https://img.shields.io/badge/iOS-13.0-green)
![Badge](https://img.shields.io/badge/Swift-5-orange)
![Badge](https://img.shields.io/badge/FirebaseAuth-blue)
![Badge](https://img.shields.io/badge/FirebaseMessaging-yellow)

![Badge](https://img.shields.io/badge/RxSwift-6.5.0-critical)
![Badge](https://img.shields.io/badge/RxCocoa-6.5.0-important)
![Badge](https://img.shields.io/badge/Alamofire-5.0.1-red)


![Badge](https://img.shields.io/badge/R.swift-6.0.1-blueviolet)
![Badge](https://img.shields.io/badge/SnapKit-5.0.1-brightgreen)
![Badge](https://img.shields.io/badge/Toast-5.0.1-ff69b4)




<br>
<br>


## 기간별 일정

`2022.01.19 - 진행중`

<br>

| 진행사항 | 진행기간 | 세부사항 |
|:---:| :--- | :--- |
| 휴대폰 인증 및 회원가입 | 2022.01.19~22.01.25 | Firebase 전화번호 인증, [SceneDelegate] UI Life Cycle 분기 처리 |
| 내 정보 | 2022.01.26~현재 | RxSwift로 TableView 구현 |
 
<br>
<br>


## 구현 이슈

<br>

<details>
<summary>버튼 활성화, `RxSwift` 적용</summary>
 
<br>

### Input/Output

ViewModel에서 입력(Input)과 출력(Output)을 정의

- View에서 받는 입력은 Input 구조체 안에 정의 (text, 버튼 이벤트)
- 로직을 통해서 나온 결과 출력은 Output 구조체에 정의 (버튼 활성화 상태, 화면 전환 이벤트)

```swift
var validText = BehaviorRelay<String>(value: "")

struct Input {
    let text: ControlProperty<String?>
    let tap: ControlEvent<Void>
}

struct Output {
    let validStatus: Observable<Bool>
    let validText: BehaviorRelay<String>
    let sceneTransition: ControlEvent<Void>
}
```
 
<br>

### 화면 전환, 비즈니스 로직 구현

- `map` 기능을 통해 정규식 유효성 검사
- `share()` 연산자를 사용하여 하나의 시퀀스에서 방출되는 아이템을 공유해 사용

```swift
func phoneNumberTransform(input: Input) -> Output {
    let result = input.text
        .orEmpty
        .map { $0.isValidPhoneNumber() }
        .share(replay: 1, scope: .whileConnected)
    return Output(validStatus: result, validText: validText, sceneTransition: input.tap)
}

func certificationTransform(input: Input) -> Output {
    let result = input.text
        .orEmpty
        .map { $0.isVaildVerificationCode() }
        .share(replay: 1, scope: .whileConnected)
    return Output(validStatus: result, validText: validText, sceneTransition: input.tap)
}
```

 <br>

- 유효성 검사가 진행되는 값을 **버튼 배경색, 버튼 활성화 상태**에 바인딩

```swift
let input = ValidationViewModel.Input(text: authView.inputTextField.rx.text, tap: authView.nextButton.rx.tap)
let output = viewModel.phoneNumberTransform(input: input)

output.validStatus
     .map { $0 ? R.color.green() : R.color.gray6() }
     .bind(to: authView.nextButton.rx.backgroundColor)
     .disposed(by: disposeBag)
        
output.validStatus
     .bind(to: authView.nextButton.rx.isEnabled)
     .disposed(by: disposeBag)

output.validText
      .asDriver()
      .drive(authView.inputTextField.rx.text)
      .disposed(by: disposeBag)

output.sceneTransition
      .subscribe { _ in
           sceneTransition()
      }.disposed(by: disposeBag)
```
 
<br>

<img src = "https://user-images.githubusercontent.com/93528918/151337727-24b9cc8d-6d4d-4479-af61-1478f22007bd.gif" width="30%" height="30%">


</div>
</details>


<br>
<br>


