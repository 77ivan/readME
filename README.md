# airbridge-ios-sdk-assignment

<br>

## Network 레이어 설계

<br>


- Endpoint
    
    - URL, path, method, parameters 등의 데이터 객체


- Provider
   
   - URLSession, DataTask를 이용하여 Network호출이 이루어 지는 곳


- APIService
   
   - URLSession을 사용하여 도메인에 종속된 Endpoint를 통해 서버에 요청

<br>

## 유실되는 이벤트 없이 서버에 전달하기

<br>

> HTTPURLResponse의 StatusCode 200을 제외한 나머지 경우
> 

<br>

- 유실된 이벤트를 담은 배열 생성

```swift
var lostEvent = [Event]()
```

<br>

- lostEvent 배열에 유실된 이벤트 append

- global Queue에서 유실된 이벤트 재요청
    - concurrent 특성을 가지기 때문에 여러 스레드로 분산되어 동시 처리

```swift
lostEvent.append(event)
                    
DispatchQueue.global().async {
    for event in lostEvent {
        track(event: event)
    }
}
```

<br>


## `exit(0)`을 고려한 멀티스레드 처리

<br>


**DispatchGroup으로 DispatchQueue들을 그룹으로 묶어서, 모든 작업이 끝난 다음 앱 종료(exit(0))**

<br>

- enter()
    
    - Dispatch Group에 들어가며, task를 +1
    
    - 선행 실행을 알림

- leave()
    
    - Dispatch Group에서 나오며, task를 -1
    
    - 실행이 끝났다는 것을 알림

- notify()
    
    - task가 0이 되었을 때 실행
    
    - 후행 실행 시작을 알림

<br>

1. 서버에 이벤트 요청 시, `enter()` 함수 호출 → 서버에 전달할 이벤트 수만큼 (for문이 순회하는만큼) 실행 알림

2. dataTask 메서드에서 response 에러가 없을 경우 디코딩 후 `leave()` 함수 호출

3. response 에러발생 시 NetworkError(.invaildResponse)를 Callback.

    global Queue에서 track 메서드 재실행 → 재실행 후 데이터를 성공적으로 보낸 뒤,`leave()` 함수 호출

4. ContentView에서 `notify(queue:)` 함수 호출 →  후행 실행 시작을 알림, **exit() 실행**


<br>

## 이벤트 전송 중 네트워크가 끊어지는 케이스

<br>


> iOS12부터는 `NWNetworkMonitor` 내부 라이브러리를 통해서 현재 인터넷 상태 변경에 대한 감지를 할 수 있도록 제공
> 

<br>

**Target OS Version 9.0**을 대응하기 위해서는 `Reachability` 라이브러리 사용

1. 네트워크 상태 모니터링을 위한 Reachability 인스턴스 생성

2. NotificationCenter에 네트워크 상태 변화를 감지하기 위한 observer 등록

3. 네트워크 상태가 변경될때마다 reachabilityChanged 메서드에서 Callback

<br>

## 이벤트 전송 중 앱이 꺼지는 케이스

<br>

- 이벤트 전송 중 background로 전환 시에는 background모드에서도 이벤트 전송 가능

- Swipe로 앱을 종료 시에는 작업 중단

<br>

## 네트워크가 불안정한 환경에서 이벤트를 전송하는 케이스


<br>

- Network Link Conditioner → **Very Bad Network** 설정 후 테스트

<br>


<img src = "https://user-images.githubusercontent.com/93528918/157194520-37d7bb1d-ff8c-4663-b367-20c555fcfe96.png" width="70%" height="70%">


<img src = "https://user-images.githubusercontent.com/93528918/157194525-ca87339d-74f6-48d8-bee9-7262aa5d8292.gif" width="70%" height="70%">




