#  TouchAd SDK  for NHPAY 설치 가이드

* 정상적인 제휴서비스를 위한 터치애드SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 TouchadSDK.xcframework, Alamofire.xcframework 폴더를 프로젝트 소스폴더내 적절히 위치시켜 줍니다.
* 앱프로젝트 target > general > Frameworks,Libraries, and Embedded Content 에서 add files 에서 TouchadSDK.xcframework, Alamofire.xcframework폴더를 선택합니다.
* Frameworks,Libraries, and Embedded Content 메뉴에서 TouchadSDK.xcframework, Alamofire.xcframework의 Embed 옵션을 ‘Embed & Sign’ 선택합니다.
* Alamofire.xcframework version 5.9.1 입니다.
* XCode 16.0 Build, Minimum Deployment 15.0 입니다.


## CocoaPods 설정
* NHPAY IOS앱은 CocoaPods를 사용하지 않습니다.


## 권한 설정
1. **광고식별자(IDFA)**
* 터치애드는 IDFA 값을 사용하여 사용자의 광고 사용 트래킹을 합니다.  
* IOS 14 이상부터 IDFA 를 사용하기 위해선 명시적으로 사용자 동의를 얻어야 합니다.
* 앱프로젝트 info.plist 에 아래내용을 추가합니다.

| Key | Type | Value |
|---|---|---|
| Information Property List|Dictionary|(1 item)|
| Privacy - Tracking Usage Description|String|앱이 타겟광고게재 목적으로 IDFA에 접근하려고 합니다.|

* 앱프로젝트에서 IDFA 조회를 명시적으로 요청할 경우 아래와 같은 메서드를 작성하여 사용합니다.

```
func requestPermission() { 
    ATTrackingManager.requestTrackingAuthorization { status in 
        switch status { 
        case .authorized: 
            print("Authorized") 
        case .denied: 
            print("Restricted") 
        @unknown default: 
            print("Unknown") 
        } 
    } 
}
```

## 터치애드 플랫폼 클래스 함수

- 주요기능을 모듈화하여 Static 함수형태로 호출합니다.
- 아래 간략한 설명입니다.
```
/**
* 포인트플러스 배너 ViewController
* @param encData: 암호화된 사용자정보 (필수)
*/
@objc public class NHPAYBannerMenuViewController: UINavigationController

/**
* 매일매일 교통적립 ViewController
* @param encData: 암호화된 사용자정보 (필수)
*/
@objc public class NHPAYApprlNoViewController: UINavigationController

/**
* 쇼핑 적립 ViewController
* @param encData: 암호화된 사용자정보 (필수)
*/
@objc public class NHPAYShoppingViewContoroller: UINavigationController
```

## 포인트플러스 화면 시작

*  NHPAY앱 내에서 포인트플러스 메뉴를 선택하면 약관동의 거치고 포인트플러스 화면을 시작할때 호출합니다.

* 아래는 포인트플러스 ViewController 호출 예시입니다.
```
var orgData = "{\"cid\"=\"123456789\",\"gender\"=\"M\",\"birthYear\"=\"1999\"}";
var encData = encrypt(orgData);

let navController = NHPAYBannerMenuViewController(encData: encData)
self.present(navController, animated:true, completion: nil)
```

## 매일매일 교통적립 화면 시작

*  NHPAY앱 내에서 매일매일 교통적립 메뉴를 선택하면 약관동의 거치고 매일매일 교통적립 화면을 시작할때 호출합니다.

* 아래는 매일매일 교통적립 ViewController 호출 예시입니다.
```
var orgData = "{\"cid\"=\"123456789\",\"gender\"=\"M\",\"birthYear\"=\"1999\"}";
var encData = encrypt(orgData);

let navController = NHPAYApprlNoViewController(encData: encData)
self.present(navController, animated:true, completion: nil)
```

## 쇼핑 적립 화면 시작

*  NHPAY앱 내에서 쇼핑 적립 메뉴를 선택하면 약관동의 거치고 쇼핑 적립 화면을 시작할때 호출합니다.

* 아래는 쇼핑 적립 ViewController 호출 예시입니다.
```
var orgData = "{\"cid\"=\"123456789\",\"gender\"=\"M\",\"birthYear\"=\"1999\"}";
var encData = encrypt(orgData);

let navController = NHPAYShoppingViewContoroller(encData: encData)
self.present(navController, animated:true, completion: nil)
```

## 빌드시  주의사항

* 애플 앱스토어 혹은 TestFlight를 통한 앱배포시에는 x86_64 아키텍쳐 빌드가 제외된 SDK 로 빌드하여야 합니다.
* arm64 빌드 SDK : 폴더/ios_touchAd_sdk/TouchadSDK.xcframework

## Sample 프로젝트

* 프로젝트명 : ios_touchAd
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
