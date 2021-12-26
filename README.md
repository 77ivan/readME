## Codable

JSON 데이터를 간편하고 쉽게 Encoding/Decoding 할 수 있게 해준다.

- Encodable+Decodable 프로토콜을 준수하는 프로토콜
- Struct, Class, Enum 모두 Codable을 채택

```swift
public typealias Codable = Decodable & Encodable
```

![Untitled](https://user-images.githubusercontent.com/93528918/147404626-4aaa9bd7-15ce-4d7a-b9e7-7bdbeb8c32dc.png)


<br>

### Codable을 이용한 Encoding

1. Codable 채택 == Decodable & Encodable 채택

```swift
struct Person: Codable {
    var name: String
    var age: Int
}
```

1. Encodable (Data → JSON)

`encode`  Person의 인스턴스를 Data타입으로 변환

encode안에 올 수 있는 값은 Encodable을 준수하고 있는 타입이어야 한다.

→ Codable 채택으로 Person타입의 인스턴스는 모두 Encodable을 준수한다 !

> throws ⇒ encoding 중 에러를 발생시킬 수 있기 때문에 try와 함께 써줘야 한다.
> 
> 
> 리턴 타입이 Data ⇒ Person 인스턴스의 데이터를 얻는 것
> 

```swift
let encoder = JSONEncoder()

let A = Person(name: "A", age: 10)

let jsonData = try? encoder.encode(A) // 인스턴스 -> Data타입
print(jsonData)
// Optional(32 bytes)
```

1. 리턴받은 Data를 json 형식으로 변환

```swift
// Data타입 -> String타입
if let prettyJsonData = jsonData,
	 let jsonString = String(data: prettyJsonData, encoding: .utf8) {
    print(jsonString)
}

// {"name":"A","age":10}
```

- 해당 코드 추가로 우리가 보던 json 형태로

```swift
encoder.outputFormatting = [.prettyPrinted, .sortedKeys]

// {
//   "age" : 10,
//   "name" : "A"
// }
```

### Codable을 이용한 Decoding

1. 이전의 Json값을 Decoding

```swift
let jsonString = """
{
  "age" : 10,
  "name" : "A"
}
"""
```

1. Decodable (JSON → Data)

`decode`  Data를 인스턴스로 변환

→ `Person.self`가 들어간 자리는 `type:` 의 자리 (Decode할 값의 타입)

![스크린샷 2021-12-26 오후 6.28.52.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d9adc1f3-3373-4d62-8644-ae8fab46643c/스크린샷_2021-12-26_오후_6.28.52.png)

```swift
let decoder = JSONDecoder()

var data = jsonString.data(using: .utf8)
print(data)
// Optional(32 bytes)

if let data = data, let myPerson = try? decoder.decode(Person.self, from: data) {
    print(myPerson)
}
// Person(name: "A", age: 10)
```

> 참고
> 

[https://zeddios.tistory.com/373](https://zeddios.tistory.com/373)

---
