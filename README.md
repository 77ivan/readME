![스크린샷 2022-02-01 오전 1 15 32](https://user-images.githubusercontent.com/93528918/151830849-c3d53fbe-6024-489b-8aa0-f74730dba198.png)


## 🌱 구현 이슈

<br>

<details>
<summary>Network 레이어 설계</summary>
 
<br>

### Network의 핵심 모듈

<br>
 
`Endpoint.`
 
- URL, path, method, parameters 등의 데이터 객체.

<br>

```swift
import Foundation

// MARK: - Method

enum Method: String {
    case GET
    case POST
    case PUT
    case DELETE
}

// MARK: - URL

enum Endpoint {
    case auth_register
    case auth_login
    case auth_password
    case post_detail_inquire(id: Int)
    case post_inquire
    case post_write
    case post_edit(id: Int)
    case post_delete(id: Int)
    case comment_inquire(postId: Int)
    case comment_write
    case comment_edit(id: Int)
    case comment_delete(id: Int)
}

extension Endpoint {
    var url: URL {
        switch self {
        case .auth_register: return .makeEndpoint("auth/local/register")
        case .auth_login: return .makeEndpoint("auth/local")
        case .auth_password: return .makeEndpoint("custom/change-password")
        case .post_detail_inquire(id: let id): return .makeEndpoint("posts/\(id)")
        case .post_inquire: return .makeEndpoint("posts?_start=0&_limit=100&_sort=created_at:desc")
        case .post_write: return .makeEndpoint("posts")
        case .post_edit(id: let id): return .makeEndpoint("posts/\(id)")
        case .post_delete(id: let id): return .makeEndpoint("posts/\(id)")
        case .comment_inquire(postId: let postId): return .makeEndpoint("comments?post=\(postId)")
        case .comment_write: return .makeEndpoint("comments")
        case .comment_edit(id: let id): return .makeEndpoint("comments/\(id)")
        case .comment_delete(id: let id): return .makeEndpoint("comments/\(id)")
        }
    }
}

extension URL {
    static let baseURL = "http://test.monocoding.com:1231/"
    
    static func makeEndpoint(_ endpoint: String) -> URL {
        URL(string: baseURL + endpoint)!
    }
}
```
 
<br>

`Provider.`

- URLSession, DataTask를 이용하여 Network호출이 이루어 지는 곳.
- Response의 타입을 알아야 제네릭을 적용할 수 있는데, 여기서 Endpoint 객체 하나만 넘기면 따로 request할 때 Response 타입을 넘기지 않아도 되게끔 설계.

<br>
  
```swift
import Foundation

extension URLSession {
    
    typealias Handler = (Data?, URLResponse?, Error?) -> Void
    
    @discardableResult
    func dataTask(_ endpoint: URLRequest, handler: @escaping Handler) -> URLSessionDataTask {
        let task = dataTask(with: endpoint, completionHandler: handler)
        task.resume()
        return task
    }
    
    static func request<T: Decodable>(_ session: URLSession = .shared, endpoint: URLRequest, completion: @escaping (T?, APIError?) -> Void) {
        
        session.dataTask(endpoint) { data, response, error in
            DispatchQueue.main.async {
                guard error == nil else { completion(nil, .failed); return }
                guard let data = data else { completion(nil, .noData); return }
                guard let response = response as? HTTPURLResponse else { completion(nil, .invaildResponse); return }
                guard response.statusCode == 200 else { completion(nil, .invaildToken); return }
                
                do {
                    let decoder = JSONDecoder()
                    let modelData = try decoder.decode(T.self, from: data)
                    completion(modelData, nil)
                } catch {
                    completion(nil, .invaildData)
                }
            }
        }
    }
}
```

 <br>



</div>
</details>


<br>

<details>
<summary>ViewModel에서 API 호출 로직 작성</summary>
 
<br>

 `ViewModel → 비즈니스 로직을 처리`
 

**ViewModel**에서 API호출하는 로직을 처리하고, **Controller**에서 알람이나 화면 전환 등 화면 처리를 해주는 걸로 이해.

❓ 그런데 아래 코드처럼 처리할 비즈니스 로직이 없는 경우, **ViewModel에서 API호출하는 코드를 작성하면 괜히 코드만 많아지는 거같아서 그냥 Controller에서 API호출을 하는 게 좋겠다는 생각**과 그래도 **MVVM을 적용한거라면 ViewModel에서 호출하는게 맞는가** 라는 생각이 듬.

<br>

![3C78364E-07BB-4C25-A823-B4188DD8A253_4_5005_c](https://user-images.githubusercontent.com/93528918/149189072-ee9a7923-11b2-4c06-aad5-171f04c2796a.jpeg)

![98287277-E478-4E1F-8FD9-7B1B0105EADD_4_5005_c](https://user-images.githubusercontent.com/93528918/149189078-a25e3cdc-97d2-4168-b398-56164ec9eb7c.jpeg)

<br>

> 멘토님 답변

결국 아키텍쳐 설계 역시 사용법, 방법론적인 것이고, 본인만의 기준을 세워 조금 변경된 패턴이나 새로운 패턴을 적용해보는 것도 아키텍처 설계에 해당.

질문의 목적을 전환해본다면 **"MVVM으로 적용하는 것이 적합할까?"**

프로젝트에서 구성된 모든 패턴이 MVVM이라고 가정한다면, 일관적인 형태로 코드의 Flow가 흘러가는 것이 중요

지금은 비즈니스 로직이 없는 뷰일지라도, 새로운 기능이 생기고, 유지보수를 하고, 여러 화면을 하나의 화면으로 합하게 될 경우 등을 고려해본다면 특정 화면만 API 호출이 뷰컨트롤러에서 이루어진다면 코드의 패턴을 파악하기가 타인이 바라볼 때는 어려울 수도 있음!



<br>

</div>
</details>


