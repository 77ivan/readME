


# 2차 제출

<br>

## TODO

- 네트워크 상태 변화, 콜백으로 에러 전달
- **앱을 재실행했을 때** 앱이 종료되기전에 보낸 데이터를 제외한 **나머지 유실된 데이터들을 처리**할 수 있도록 구현
- 유효하지 않은 응답을 받은 이벤트를 다시 처리할 경우, 고객사 앱에 부담을 주지않는 방향과 재호출 시간 간격을 제어하여 또 다시 유실될 확률 줄이기

<br>

## 네트워크 변경 감지

> ISSUE

- NotificationCenter에 네트워크 상태 변화를 감지하기 위한 observer 등록하여 네트워크 상태가 변경될때마다 `reachabilityChanged` 메서드에서 Callback
    - `reachabilityChanged`(**#selector**)는 매개변수를 2개를 사용할 수 없기 때문에, 네트워크 에러를 completion 구문으로 넘겨주지 못한다.


> Solution

- **클로저 구문**으로 네트워크가 연결되지 않았을 때, completion 구문으로 에러 전달
    - API 호출 코드 블록(Provider)에 NetworkMoniter를 연결해줌으로써, API를 호출할 때마다 네트워크 연결 상태 체크



<details>
<summary> (전) NetworkMoniter 코드 </summary>
<div markdown="1">
<br>

- 콜백으로 에러구문을 전달하지 못해서 사용자가 에러 여부를 알 수 없다.

<br>
    
```swift
let reachability = try! Reachability()

func startMonitoring() {
    NotificationCenter.default.addObserver(
        self,
        selector: #selector(reachabilityChanged),
        name: Notification.Name.reachabilityChanged,
        object: reachability
    )

    do {
        try reachability.startNotifier()
    } catch {
        print("Could not start reachability notifier")
    }
}

/// 네트워크 status가 변경될 때마다 호출
@objc func reachabilityChanged(notification: Notification) {
    let reachability = notification.object as! Reachability

    switch reachability.connection {
    case .wifi, .cellular:
        print("network connected!")
    case .unavailable:
        print(NetworkError.internet)
    }
}
```
    
    
</div>
</details>












https://user-images.githubusercontent.com/93528918/159855014-4f2870e8-def4-4345-9f7d-1047e75fa8e7.mov




https://user-images.githubusercontent.com/93528918/159855125-e67262bf-45d2-42fd-b6d3-8c6cb4b3dd2b.mov






