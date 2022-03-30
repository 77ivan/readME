
# 서비스 레벨 프로젝트

- 서비스 레벨의 API/기획 명세와 디자인 리소스를 바탕으로 iOS 앱개발

- 휴대폰 인증 및 회원가입을 통해 앱에 로그인을 진행.

- Map에서 사용자의 위치를 확인하고, 다른 사용자들과 취미를 공유할 수 있도록 매칭이 가능한 앱


<br>

## Tech & Tool

- Swift, MVVM, Rest API, Mapkit
- AutoLayout, StoryBoard, Code Base UI
- Xcode, Figma, SwaggerUI, Confluence

<br>

## Library

- [FirebaseAuth, FirebaseMessaging](https://firebase.google.com/docs/ios/setup?hl=ko)
- [RxSwift, RxCocoa](https://github.com/ReactiveX/RxSwift)
- [Alamofire](https://github.com/Alamofire/Alamofire)
- [Moya](https://github.com/Moya/Moya)
- [R.swift](https://github.com/mac-cain13/R.swift)
- [SnapKit](https://github.com/SnapKit/SnapKit)
- [Toast](https://github.com/scalessec/Toast-Swift)
- [Tabman](https://github.com/uias/Tabman)
- [RangeSeekSlider](https://github.com/WorldDownTown/RangeSeekSlider)


<br>

## View

### 로그인

- **Page Control**를 사용한 온보딩 페이지
- 전화번호 입력, 인증번호 입력 TextField에 **[.phonePad] 키보드 타입 적용**
- **Firebase**를 활용한 전화번호 인증 프로세스 개발
- NSPredicate(정규식)을 통해 전화번호, 인증번호의 유효성을 검사한 뒤, **버튼 활성화**
- 인증 번호 확인 절차에서 **타이머 기능과 재전송 기능** 구현

![스크린샷 2022-03-30 오후 10 18 31](https://user-images.githubusercontent.com/93528918/160843798-2c1bc17e-ad5a-4e07-9fd9-30af575fae1d.png)


<br>

### 회원가입

- 전화번호 인증 후, 닉네임 / 이메일 / 생년월일 / 성별을 기입하여 회원가입
- NSPredicate(정규식)을 통해 닉네임 / 이메일의 유효성을 검사한 뒤, **버튼 활성화**
- 생년월일은 현재 날짜 기준 **만 17세 이상 여부 판단**하여 버튼 활성화
- **성별은 선택하지않아도** 회원가입 가능하도록 구현 (애플 심사규정 고려)


![스크린샷 2022-03-30 오후 10 21 09](https://user-images.githubusercontent.com/93528918/160844289-9e61c2ea-9b44-47e3-8b10-8da57f0be6bf.png)



<br>

### 마이 페이지

- 정보 관리 페이지는 여러가지 Cell Type으로 하나의 TableView 구성
- 사용자의 이름이 입력된 Cell은 **확장 View**로 구성
- 각 Cell의 데이터 편집 후, 저장버튼을 통해 수정 가능


![스크린샷 2022-03-30 오후 10 21 38](https://user-images.githubusercontent.com/93528918/160844393-f3e35e30-5d6e-4d34-8e41-571b2fd6401e.png)


<br>


### 홈

- **MapKit**을 통해 다른 사용자의 정보와 위치를 확인
- **성별 필터 버튼**을 통해 사용자들 필터링 기능
- **GPS 버튼**을 통해 자신의 현재 위치로 이동
- 우측 하단의 **플로팅 버튼**을 통해 취미를 찾거나, 매칭된 사람과 채팅을 하거나, 매칭 되기를 기다릴 수 있음
- 위치 거부 상태일 경우, 지정해둔 위치로 이동

<img src = "https://user-images.githubusercontent.com/93528918/160844489-57b5d366-8897-4f15-9215-dc6477369e4f.gif" width="30%" height="30%">

<br>

### 취미 입력 화면

- 서버에서 추천하는 취미(빨간색 버튼)과 현재 위치 기준으로 주변의 사용자들이 서버로 보낸 취미(회색 버튼)은 첫번째 Section에 배치
- 내가 하고 싶은 취미(초록색 버튼)은 두번째 Section에 배치
- Search Bar를 통해 취미를 추가할 수 있고, 띄어쓰기를 통해 복수 입력이 가능
- 첫번째 Section의 Cell을 선택하면 두번째 Section으로 이동
- 두번째 Section을 선택하면 “내가 하고싶은” 취미 목록에서 삭제
- “내가 하고싶은” 취미 목록은 **최대 8개로 제한**하고, **동일한 취미가 들어가지 못하도록 제한**
- “내가 하고싶은” 취미를 통해 주변 사용자들에게 **매칭을 기다릴 수 있는 상태**로 변경됨 (플로팅 이미지 변경)
- “찾기 중단” 버튼을 누르면 **일반 상태**로 변경됨 (플로팅 이미지 변경)

<img src = "https://user-images.githubusercontent.com/93528918/160844729-a27da727-6076-4ec3-838c-37d0ba365a8d.gif" width="30%" height="30%">


<br>





## 구현 이슈

<br>

<details>
<summary>Moya 도입, 네트워크 구조 개선</summary>
 
 <br>
 
### [Moya 도입] 전
 
 > Endpoint, APIService에 모든 네트워크 통신에 대한 메서드를 다 구현
 
  <br>
 
 <details>
<summary>Endpoint - URL</summary>
 
 <br>

  ```swift
enum Endpoint {
    case user
    case user_withdraw
    case user_update_fcm_token
    case user_update_mypage
  
    case queue
    case queue_onqueue
    ...
}

extension Endpoint {
    var url: URL {
        switch self {
        case .user: return .makeEndpoint("user")
        case .user_withdraw: return .makeEndpoint("user/withdraw")
        case .user_update_fcm_token: return .makeEndpoint("user/update_fcm_token")
        case .user_update_mypage: return .makeEndpoint("user/update/mypage")
  
        case .queue: return .makeEndpoint("queue")
        case .queue_onqueue: return .makeEndpoint("queue/onqueue")
        ...
        }
    }
}

extension URL {
    static let baseURL = "http://test.monocoding.com:35484/"

    static func makeEndpoint(_ endpoint: String) -> URL {
        URL(string: baseURL + endpoint)!
    }
}
```
<br>
  
 </div>
</details>

<br>


 <details>
<summary>APIService - HTTPHeaders, Parameters, Request</summary>

<br>

 ```swift
import Alamofire

...

static func signUpUserInfo(idToken: String, completion: @escaping (Error?, Int?) -> Void) {
        
    let headers: HTTPHeaders = [
        "idtoken": idToken,
        "Content-Type": "application/x-www-form-urlencoded"
    ]
        
    let FCMtoken = UserDefaults.standard.string(forKey: "FCMToken") ?? ""
    let phoneNumber = UserDefaults.standard.string(forKey: "phoneNumber") ?? ""
    let nick = UserDefaults.standard.string(forKey: "nickName") ?? ""
    let birth = UserDefaults.standard.string(forKey: "birth") ?? ""
    let email = UserDefaults.standard.string(forKey: "email") ?? ""
    let gender = UserDefaults.standard.integer(forKey: "gender")
        
    let parameters : Parameters = [
        "phoneNumber": phoneNumber,
        "FCMtoken": FCMtoken,
        "nick": nick,
        "birth": birth,
        "email": email,
        "gender": gender
    ]
        
    AF.request(Endpoint.user.url.absoluteString, method: .post, parameters: parameters, headers: headers).responseString { response in
            
        let statusCode = response.response?.statusCode
            
        switch response.result {
        case .success(let value):
            print("[signUpUserInfo] response success", value)
            completion(nil, statusCode)
                
        case .failure(let error):
            print("[signUpUserInfo] response error", error)
            completion(error, statusCode)
        }
    }
}
 
...
 
```
 
<br>
  
 </div>
</details>
 
<br>
<br>

 
### [Moya 도입] 후
 
 > API에도 목적이 존재하는 만큼 자체적인 기준을 세워서 역할/책임을 조금 더 분리 필요.
 > 
 > 이후에 서버와 커뮤니케이션을 할 때, 용이하거나 변경 지점이 생기시더라도 금방 유지보수가 가능
 
 <br>


Target (baseURL, path, method, task, headers)

- [UserTarget](https://github.com/camosss/SeSAC_SPL/blob/main/SeSAC_SPL/Network/User/APIs/UserTarget.swift)

- [QueueTarget](https://github.com/camosss/SeSAC_SPL/blob/main/SeSAC_SPL/Network/Queue/APIs/QueueTarget.swift)

 <br>

API (request)

- [UserAPI](https://github.com/camosss/SeSAC_SPL/blob/main/SeSAC_SPL/Network/User/APIs/UserAPI.swift)

- [QueueAPI](https://github.com/camosss/SeSAC_SPL/blob/main/SeSAC_SPL/Network/Queue/APIs/QueueAPI.swift)

<br>

Models (Request body)

- [User_Models](https://github.com/camosss/SeSAC_SPL/tree/main/SeSAC_SPL/Network/User/APIs/Models)

- [Queue_Models](https://github.com/camosss/SeSAC_SPL/tree/main/SeSAC_SPL/Network/Queue/APIs/Models)


<br>



 </div>
</details>


<br>

<details>
<summary>[MVVM 패턴] 여러가지 Cell Type으로 하나의 TableView 구성하기</summary>
 
 <br>
 
[블로그 포스팅으로 이동하기](https://llan.tistory.com/3)

 </div>
</details>


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

<br>
<br>

</div>
</details>

<br>

<details>
<summary>[SceneDelegate] 로그인/회원가입 유무에 따른 UI Life Cycle 분기 처리</summary>
 
<br>

 ### 첫번째 시도.
 
 1. 회원정보를 앱내 스토리지(저장소)에 저장해두고 필요할때 불러와서 처리하기 위해 토큰 값을 UserDefaults에 저장.
 2. 로그인과 회원가입 분기처리는 로그인 여부에 달려있기에, 서버로부터 로그인 시 발급받은 토큰을 SceneDelegate에서 앱 실행 시에 토큰 유무에 따라 UI Life Cycle 분기 처리

 <br>
 
 > idToken 값으로 분기 처리를 하기 위해, User의 정보를 API에서 호출했는데 API에서 데이터를 받아오는 과정에서 black Screen이 뜬 뒤, View가 로드된다.
 
 
   <br>

 <details>
 <summary>코드</summary>
 
  <br>

```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
    guard let scene = (scene as? UIWindowScene) else { return }
    window = UIWindow(windowScene: scene)

    let idToken = UserDefaults.standard.string(forKey: "idToken") ?? ""
    print("SceneDelegate idToken", idToken)

    if idToken == "" { // 전화번호 인증 X
        convertNavRootViewController(VerificationViewController())
    } else { // 전화번호 인증 O
        APIService.getUserInfo(idToken: idToken) { user, error, statusCode in
            switch statusCode {
            case 200:
                self.convertRootViewController(MainTapController())

            case 401:
                print("SceneDelegate", statusCode ?? 0)
                Helper.getIDTokenRefresh {
                    print("SceneDelegate 토큰 갱신 error"); return
                } onSuccess: {
                    print("SceneDelegate 토큰 갱신 성공")
                    self.convertRootViewController(MainTapController())
                }

            default:
                print("SceneDelegate default error", statusCode ?? 0)
                self.convertNavRootViewController(NickNameViewController())
            }
        }
    }
}
```
   </div>
 </details>
  <br>
 
<img src = "https://user-images.githubusercontent.com/93528918/151345005-9918e493-9e83-46ec-a4f5-fca9f2953a70.gif" width="30%" height="30%">

  <br>

  

 <br>
 <br>
 
### 두번째 시도.

 1. 로그인 완료
 2. 회원가입 완료
 3. 회원 탈퇴 완료

    <br>

 > 굳이 API 호출을 하지 않고 3가지의 상황에 따라 UserDefaults에 상황별 String값을 저장해주고, SceneDelegate에서 해당 Key값을 통해 UI Life Cycle 분기 처리 진행
 
 <br>


 <details>
 <summary>코드</summary>
 
   <br>

 ```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        
    guard let scene = (scene as? UIWindowScene) else { return }
    window = UIWindow(windowScene: scene)

    let startView = UserDefaults.standard.string(forKey: "startView")
    print("------> startView = \(startView ?? "전화번호인증 하러가야함")")
        
    if startView == "successLogin" { // 로그인 완료
        convertNavRootViewController(NickNameViewController())
    } else if startView == "alreadySignUp" { // 회원가입 완료
        convertRootViewController(MainTapController())
    } else { // 회원탈퇴 완료 및 앱 첫 실행
        convertNavRootViewController(VerificationViewController())
    }
   
}
```
  </div>
 </details>
  <br>

 <img src = "https://user-images.githubusercontent.com/93528918/151345253-295ddc6c-9250-43a9-a717-6b29574e8bee.gif" width="30%" height="30%">

  <br>

 
 
</div>
</details>
 
<br>
<br>


