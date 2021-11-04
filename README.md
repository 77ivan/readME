## 백업, 복구

### 백업

1. 데이터가 저장된 document 위치 찾기

```swift
func documentDirectoryPath() -> String? {
    let documentDirectory = FileManager.SearchPathDirectory.documentDirectory
    let userDomainMask = FileManager.SearchPathDomainMask.userDomainMask
    let path = NSSearchPathForDirectoriesInDomains(documentDirectory, userDomainMask, true)
        
    if let directoryPath = path.first {
        return directoryPath
    } else {
        return nil
    }
}
```

2. 백업할 파일 주소(**/default.realm**)를 추가하고, 파일 존재 여부를 확인한 뒤에 URL배열 (백업할 파일에 대한 URL배열) 에 추가한다.

```swift
Users/camosss/Library/Developer/ ... /Documents/default.realm
```

3. 압축 진행
- `Zip` 프레임워크를 사용해서 백업할 파일 압축을 진행한다.
- UIActivityViewController를 통해 백업을 완료한 파일을 공유할 수 있다!

```swift
do {
    // 압축 경로
    let zipFilePath = try Zip.quickZipFiles(urlPaths, fileName: "archive") // Zip
    presentActivityViewController()
} catch {
     print("Something went wrong")
}
```

![스크린샷 2021-11-04 오후 7 29 45](https://user-images.githubusercontent.com/93528918/140302351-101359ac-b60f-4fd4-b03b-12f1980fd61d.png)


https://user-images.githubusercontent.com/93528918/140302304-6148ed63-384c-4283-8093-2daf7d141ccd.mov


---

### 복구

- 아이폰의 `파일` 앱을 가져오기 위해 `MobileCoreServices` 임포트
- UIDocumentPickerViewController를 delegate로 연결한다.
- 선택한 파일의 경로를 가져와서 압축 해제 (이 과정 또한 파일의 존재 여부를 확인한다.)
- `Zip` 프레임워크의 압축해제 코드

**destination: 위치, overwrite: 덮어쓰기, progress: 진행상황**

```swift
try Zip.unzipFile(fileURL, destination: documentDirectory, overwrite: true, password: nil,
    progress: { progress in
        // 복구가 완료됨을 알림
    }, fileOutputHandler: { unzippedFile in
        print("unzippedFile \(unzippedFile)")
    })
```

- 파일이 해당 document에 저장되어있지 않다면, document 폴더에 옮겨준다.

```swift
try FileManager.default.copyItem(at: selectedFileURL, to: sandboxFileURL)
```


https://user-images.githubusercontent.com/93528918/140302312-63587b81-ecde-4101-80d2-250a3ebb8a80.mov


---

cf. **SandBox**

> SandBox란 커널 수준에서 강제 적용되는 맥 OS의 접근 제어 기술이다.
> 사용자의 파일앱에 저장하는 방법을 알기전에, 샌드박싱의 개념을 알고가야 한다.

> 예를 들어, 하나의 폰에서 실행중인 두개의 앱을 실행중이다. 그런데, 하나의 앱에 악성 소프트웨어가 있는데 같은 폰에 있는 다른 앱을 감염시키려 한다. iOS에서는 이 문제를 샌드박싱으로 해결한다.

> 폰의 모든 앱이 자체 샌드박스 안에 있다고 해보자. 샌드박스는 보호된 환경 그 이상도 아니다. 또는 앱을 위한 작은 감옥으로 생각할 수도 있다.

> 각 앱에는 앱과 관련된 파일 및 문서를 저장하는 자체 폴더가 있다. 따라서 앱이 데이터를 저장하거나 검색해야 할 때 데이터를 읽고 쓸 수 있다. 해당 문서 폴더에 액세스할 수 있지만, 다른 앱 문서 폴더에 엑세스할 수는 없다.

> 대신 폰이 iCloud와 동기화되거나 laptop에 연결할 때마다 발생한다. 예를 들어, 폰을 새로 구입하는 경우 iCloud 또는 iTunes와 지속적으로 동기화되기 때문에 문서 폴더에 저장한 모든 데이터는 삭제되지 않는다.

> 또한, 앱 자체가 운영 체제의 코드에 영향을 줄 수 있음을 의미하기에, 운영 체제에서 보안 데이터를 가져오기 위해악성 앱을 작성할 수 없다. 예를 들어, 테이블쪽으로 폰을 놓으면 방해금지모드로 전환되는 앱, iOS에서는 이것을 구현할 방법이 없다.
