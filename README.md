<details>
<summary>Moya 도입, 네트워크 구조 개선</summary>
 
 <br>
 
 [Moya 도입] 전
 
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
}

extension Endpoint {
    var url: URL {
        switch self {
        case .user: return .makeEndpoint("user")
        case .user_withdraw: return .makeEndpoint("user/withdraw")
        case .user_update_fcm_token: return .makeEndpoint("user/update_fcm_token")
        case .user_update_mypage: return .makeEndpoint("user/update/mypage")
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
  
  
 </div>
</details>
 
 [Moya 도입] 후
 
 > API에도 목적이 존재하는 만큼 자체적인 기준을 세워서 역할/책임을 조금 더 분리 필요.
 > 이후에 서버와 커뮤니케이션을 할 때, 용이하거나 변경 지점이 생기시더라도 금방 유지보수가 가능
 
 <br>


 </div>
</details>
