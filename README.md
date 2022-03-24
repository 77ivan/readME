# airbridge-ios-sdk-assignment

## Index

- [1차제출](#1차제출)

- [2차제출](#2차제출)

<br>
<br>

# 1차제출

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




<br>
<br>
<br>



# 2차제출

<br>

## TODO



- 네트워크 상태 변화, 콜백으로 에러 전달

- **앱을 재실행했을 때,** 앱이 종료되기 전에 보낸 데이터를 제외한 **나머지 유실된 데이터들을 처리**할 수 있도록 구현

- 유효하지 않은 응답을 받은 이벤트를 다시 처리할 경우

	- 고객사 앱에 부담을 주지않는 방향
	
	- 재호출 시간 간격을 제어하여 또 다시 유실될 확률 줄이기

<br>

## 네트워크 변경 감지

<br>

> ISSUE

- 1차 제출 → NotificationCenter에 네트워크 상태 변화를 감지하기 위한 observer 등록하여 네트워크 상태가 변경될때마다 `reachabilityChanged` 메서드에서 Callback
    
    - `reachabilityChanged`(**#selector**)는 매개변수를 2개를 사용할 수 없기 때문에, 네트워크 에러를 completion 구문으로 넘겨주지 못한다.


<br>

> Solution

- **클로저 구문**으로 네트워크가 연결되지 않았을 때, completion 구문으로 에러 전달
    
    - API 호출 코드 블록(Provider)에 NetworkMoniter를 연결해줌으로써, API를 호출할 때마다 네트워크 연결 상태 체크


<br>

> Code

<br>

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

<br>

- [(후) NetworkMoniter 코드](https://github.com/AB180-HR/220303-KangHosung/blob/master/assignment-sdk/source/network/service/NetworkMoniter.swift)

- [API 호출 구문 코드 (Provider)](https://github.com/AB180-HR/220303-KangHosung/blob/master/assignment-sdk/source/network/layer/Provider.swift)



<br>

## struct → class (AssignmentSDK)

<br>

> ISSUE


- Event를 담은 배열을 메서드안에 생성하여 **하나의 Event**를 담은 배열이 반복적으로 메모리에 올라간다.

    - 하나의 **event 배열**에 저장하지 못한다.

<br>

> Solution


- 저장해야하는 배열이 메서드 블록이 아닌 프로퍼티로 빠져야한다.

    - 구조체(struct)가 멤버 프로퍼티를 업데이트하려면 mutating 속성을 붙여야 한다.
    
    - 하지만 클로저가 캡쳐하는 시기가 클로저가 생성될 때인데, APIService에서 completion으로 `@escaping closure`를 사용하고 있고, 거기에서 self를 캡처한다.
    
        - 그 시기는 `@escaping closure`가 생성될 때라고 할 수 있는데, 만약 self가 mutating된다면 `@escaping closure` 입장에서는 잘못된 self를 캡처하게 될 수 있어서 mutating self를 캡처하고 있다고 오류가 난다.
        
        - Error: Escaping closure captures mutating 'self' parameter
        
    - 클로저가 클래스(class)를 캡처할 때는 실행 시 캡처가 이루어지고, 참조타입이기 때문에 class의 속성이 변경이 되어도 `@escaping closure`는 자신이 의도한 인스턴스가 참조가 된다.

<br>

## 앱 종료로 유실된 이벤트 처리하기

<br>

> ISSUE


- 1차 제출 → DispatchGroup으로 DispatchQueue들을 그룹으로 묶어서, 모든 작업이 끝난 다음 앱 종료

    - view에서 DispatchGroup을 생성 후, notify로 콜백을 받는 코드까지 작성해야하는 불편함이 있다.

    - 모든 작업이 끝나기를 기다리고 종료되야하는 상황에서 사용자가 앱을 강제종료(스와이프로)한다면 유실된 이벤트를 처리할 방법이 없다.

<br>

- 피드백

    - sdk를 제공하는 것이기 때문에, 고객사에서 추가작업 없이 sdk를 사용할 수 있도록 고려
    
    - exit()을 고려해서 데이터를 다 보내고 종료하는 것이 아닌, 앱을 재실행했을 때 앱이 종료되기전에 보낸 데이터를 제외한 나머지 유실된 데이터들을 처리할 수 있도록 구현

<br>

> Solution


- UserDefaults를 활용하여 앱 종료로 유실된 object(event) 배열을 **스토리지에 저장**

<br>

---

**1차 시도**

- view에 sdk를 인스턴스로 할당할 때, **sdk 내부 초기화 구문**을 통해 앱 종료로 유실된 이벤트 불러와서 유실된 이벤트가 있는지 Bool(isLost)로 구분

    - track 메서드 내부에서 유실된 이벤트를 반복적으로 불러오는 건 비효율적이기 때문에 초기화 구문에서 구현


- view에서 **track메서드 호출** 시, 초기화 구문에서 체크한 Bool(isLost)값을 활용하여 유실된 이벤트 배열을 처리하는 조건문을 생성
    - 유실된 이벤트 배열과 새로들어오는 이벤트 배열을 동시에 처리


<br>

<details>
<summary> 1차 시도 코드 </summary>
<div markdown="1">
<br>


    
```swift
public init() {
    /// sdk를 할당할 때, 앱 종료로 유실된 이벤트 불러와서 Bool(flag)로 구분
    let remainEvent = PersistenceManager.populateEvent()
    isLost = remainEvent.isEmpty ? false : true
}

public func track(
    event: Event,
    onCompletion: @escaping (NetworkError?) -> Void
) {
    if isLost { /// 앱 종료로 유실된 이벤트 처리
        handleRemainEvent(onCompletion: onCompletion)
        isLost = false
    }

	/// 새로 들어온 이벤트 처리
	  ....
}

private func handleRemainEvent(
    onCompletion: @escaping (NetworkError?) -> Void
) {
    let remainEvent = PersistenceManager.populateEvent()

    remainEvent.forEach { event in
        handlePostEvent(event: event, onCompletion: onCompletion)
    }
}
```
    
    
</div>
</details>

<br>

**문제점**

- sdk 내부 초기화 구문에서 유실 여부 체크후 track 메서드 실행할 때, 유실된 이벤트를 비동기로 처리하는데, 새로운 이벤트에서 또 유실이 발생한다면 앱을 재실행할 때마다 **새로운 allEvent 배열이 생성**되서 유실된 이벤트들이 쌓이지 않는다.

- 데이터를 수집하는 목적으로 봤을 때, 앱을 재실행 후 유실된 데이터들을 처리하기 위해 track 메서드를 호출해야 처리되는 로직
   
   - **유실된 데이터가 있는 상태에서 track 메서드를 호출하지 않는다는 가정**에서는 끝까지 유실된 채로 유지되기 때문에 좋지 못한 방법이라고 생각



<br>

---

**2차 시도**

- **sdk 내부 초기화 구문에서 유실된 이벤트 처리하는 로직을 넣음**으로써, 앱을 재실행했을 때 유실된 이벤트가 남아있다면 자동으로 처리되도록 구현

- 유실 여부 체크하는 코드를 제거하고, 앱을 재실행할 때 유실된 이벤트 처리하기 때문에 새로운 **allEvent 배열**이 누적되지 못하는 문제를 고려할 필요가 없어진다.

<br>


<details>
<summary> 2차 시도 코드 </summary>
<div markdown="1">
<br>


    
```swift
public init() {
    /// sdk를 할당할 때, 앱 종료로 유실된 이벤트 처리
    let remainEvent = PersistenceManager.populateEvent()
    handleRemainEvent(remainEvent: remainEvent, onCompletion: nil)
}

private func handleRemainEvent(
    remainEvent: [Event],
    onCompletion: ((NetworkError?) -> Void)?
) {
    remainEvent.forEach { event in
        handlePostEvent(event: event, onCompletion: onCompletion)
    }
}
```

    
    
</div>
</details>

<br>

---

<br>
<br>


## 유효하지않은 응답값의 이벤트 처리

<br>

> ISSUE

1. 해당 이벤트가 유효하지 않은 응답을 받았을 때, 바로 재호출을 하게 된다면 또 다시 유실될 확률이 높다.

2. 1차 제출 → 유실된 이벤트가 발생하지 않도록 무한 재귀 호출

    - 데이터 수집의 sdk 측면에서는 정확히 빠짐없이 데이터를 전달해야 한다.
    
    - 하지만 1, 2개의 이벤트가 계속해서 유실되어 해당 이벤트를 처리할 때까지 API 호출이 진행될 경우, 고객사 리소스를 잡아먹으면서 품질을 저하시키게 된다.

<br>

> Solution


1. 유실된 이벤트를 재호출하는 시간을 2의 배수로 점차 늘리면서 유실될 확률을 줄임

2. 고객사 앱에 부담을 주지 않는 방향(경량화 측면)으로 데이터를 일부 포기하기 위해 DispatchSemaphore를 활용하여 재호출 가능한 자원의 갯수를 제어
    
    - 데이터는 통계적이기 때문에 1, 2개의 유실된 이벤트가 크게 영향을 미치지 않는다고 생각


<br>

## 의문점

<br>


> API 호출 로직에서 콜백으로 에러 전달


- 데이터를 수집하는 sdk 측면에서 **API 호출 로직(sdk 내부)에서 발생하는 에러**를 사용자에게 보여주는 것에 대한 의문

- **사용자의 유입이나 라이플사이클을 파악하기 위한 이벤트를 수집하는 sdk**라면 **이벤트 유실 여부**나 **API 호출 실패**에 대한 case는 앱을 이용하는 사용자에게 보여줘야할 필요성이 있는 것인지에 대한 의문


<br>
<br>

> 앱 종료로 스토리지에서 유실된 이벤트를 불러올 때, 빈 배열이 들어오는 케이스


1. 앱 종료로 유실된 object(event) 배열을 스토리지에 저장

2. 앱 재실행

3. sdk가 인스턴스로 할당되어있는 view에서 sdk 내부 초기화 구문을 통해 유실된 이벤트들을 스토리지에서 불러오기

<br>

- 앱이 종료되기 전에 **스토리지에 저장을 마친 후,** API 로직에 들어가지 못하고 앱이 종료된다.

	- 앱을 재실행하고 초기화 구문에서 스토리지에 저장된 이벤트 배열을 불러올 때, 저장된 배열을 불러오지 못하는 case가 발생

	- 앱 종료 전에 스토리지에 저장된 시점까지 확인이 가능한데, 재실행했을 때 빈 배열이 들어오는 case에 대한 의문


- 빈 배열이 들어온다는 건 스토리지에 저장된 배열을 불러오는 과정에서 빈 배열이 return이 되는 것

    - **PersistenceManager.populateEvent**로직에서 event 키값에 저장된 배열이 없다.
    
    <br>
    
    <details>
	<summary> PersistenceManager.populateEvent </summary>
	<div markdown="1">
	<br>

	```swift
	static func populateEvent() -> [Event] {
   	 guard let events = defaults.object(forKey: Token.event) as? Data
   	 else { return [] }
	
   	 do {
    	    let decoder = JSONDecoder()
      	  let domainsSchema = try decoder.decode([Event].self, from: events)
      	  return domainsSchema
    	} catch {
    	    return([])
   	 }
	}
	```
	
	<br>

	https://user-images.githubusercontent.com/93528918/159867208-15310a0d-e9d5-4ad7-918c-670d50167c45.mov

	<br>

   	 
	</div>
	</details>



	<br>



<br>

**성공 case**

https://user-images.githubusercontent.com/93528918/159855014-4f2870e8-def4-4345-9f7d-1047e75fa8e7.mov


<br>


**실패 case**

https://user-images.githubusercontent.com/93528918/159855125-e67262bf-45d2-42fd-b6d3-8c6cb4b3dd2b.mov



<br>

<details>
<summary> 디버깅 구간 </summary>
<div markdown="1">
<br>

```swift
public init() {
    let remainEvent = PersistenceManager.populateEvent()
		print("스토리지에 저장된 유실이벤트", remainEvent)
    handleRemainEvent(remainEvent: remainEvent, onCompletion: nil)
}

public func track(
    event: Event,
    onCompletion: ((NetworkError?) -> Void)?
) {
    allEvent.append(event)
    PersistenceManager.saveEvent(events: allEvent) /// 호출 이벤트 스토리지에 저장
    
print("스토리지에 저장", PersistenceManager.populateEvent())

    handlePostEvent(event: event, onCompletion: onCompletion)
}
```

    
    
</div>
</details>




<br>
<br>














