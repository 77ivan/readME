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
