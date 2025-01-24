#  TouchAd SDK for DIGILOCA 설치 가이드

* Touchad 광고플랫폼 제휴서비스를 위한 TouchadSDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 TouchadSDK.xcframework 폴더를 프로젝트 소스폴더내 적절히 위치시켜 줍니다.
* 앱프로젝트 target > general > Frameworks,Libraries, and Embedded Content 에서 add files 에서 TouchadSDK.xcframework, Alamofire.xcframework 폴더를 선택합니다.
* Frameworks,Libraries, and Embedded Content 메뉴에서 TouchadSDK.xcframework의 Embed 옵션을 ‘Embed & Sign’ 선택합니다.
* Xcode 16.1 Build, Minimum Deployment 14.0 입니다.


## CocoaPods 설정
* DIGILOCA IOS앱은 CocoaPods를 사용하지 않습니다.

## 권한 설정

1. **광고식별자(IDFA)**
* Touchad는 IDFA 값을 사용하여 사용자의 광고 사용 트래킹을 합니다.  
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

## 개인정보 보호 매니페스트 파일(Privacy manifest file)
1. **PrivacyInfo.xcprivacy**
* 2024년 5월 1일부터 특정 API를 사용할 경우 허용된 사유가 포함된 PrivacyInfo.xcprivacy 파일이 프로젝트에 포함되어야 합니다.
* 터치애드 SDK 산출물내 포함된 PrivacyInfo.xcprivacy에 아래내용을 추가했습니다.
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>NSPrivacyTracking</key>
    <false/>
    <key>NSPrivacyTrackingDomains</key>
    <array/>
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeDeviceID</string>
            <key>NSPrivacyCollectedDataTypeLinked</key>
            <false/>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
            <key>NSPrivacyCollectedDataTypePurposes</key>
            <array>
                <string>NSPrivacyCollectedDataTypePurposeThirdPartyAdvertising</string>
            </array>
        </dict>
    </array>
    <key>NSPrivacyAccessedAPITypes</key>
    <array>
        <dict>
            <key>NSPrivacyAccessedAPIType</key>
            <string>NSPrivacyAccessedAPICategoryUserDefaults</string>
            <key>NSPrivacyAccessedAPITypeReasons</key>
            <array>
                <string>CA92.1</string>
            </array>
        </dict>
    </array>
</dict>
</plist>
```

## Touchad 플랫폼 public 클래스

- 주요기능화면을 제어할 수 있게 UIViewController 확장클래스를 이용합니다.
- 아래 간략한 설명입니다.
```
/**
* 하루세번머니
* @param isProd: 개발 / 상용 도메인 구분 값(true = 상용, falase = 개발)
* @param isModal: 모달 or 스택 화면노출 방식 여부 값(true = 모달, false = 스택)
* @param cid: 디지로카 회원관리번호
* @param gender: 회원 성별(ex: 남성 = M, 여성 = F)
* @param birthYear: 회원 출생년도(ex: 1992)
*/
func openTodayEarningMenu(isProd: Bool, isModal: Bool = false, cid: String, gender: String, birthYear: String)

/**
* 라방보고머니
* @param isProd: 개발 / 상용 도메인 구분 값(true = 상용, falase = 개발)
* @param isModal: 모달 or 스택 화면노출 방식 여부 값(true = 모달, false = 스택)
* @param cid: 디지로카 회원관리번호
* @param gender: 회원 성별(ex: 남성 = M, 여성 = F)
* @param birthYear: 회원 출생년도(ex: 1992)
*/
func openLabangEarningMenu(isProd: Bool, isModal: Bool = false, cid: String, gender: String, birthYear: String)

```


## 하루세번머니 화면 시작

* 디지로카 앱 내에서 하루세번머니 메뉴를 선택하면 약관동의 거치고 하루세번머니 화면을 시작할때 호출합니다.

* 아래는 하루세번머니 화면 시작함수 호출 예시입니다.

* Swift
```
let isProd = true(상용) or false(개발)
let isModal = true(모달) or false(스택)
TASDKManager.openTodayEarningMenu(isProd: isProd, isModal: isModal, cid: "회원관리번호", gender: "M", birthYear: "1996")
```

* Objective-C
```
BOOL isProd = YES(상용) or NO(개발);
BOOL isModal = YES(모달) or NO(스택);
[TASDKManager openTodayEarningMenuWithIsProd:isProd isModal:isModal cid:@"회원관리번호" gender:@"M" birthYear:@"1996"];
```

## 라방보고머니 화면 시작

* 디지로카 앱 내에서 라방보고머니 메뉴를 선택하면 약관동의 거치고 라방보고머니 화면을 시작할때 호출합니다.

* 아래는 라방보고머니 화면 시작함수 호출 예시입니다.

* Swift
```
let isProd = true(상용) or false(개발)
let isModal = true(모달) or false(스택)
TASDKManager.openLabangEarningMenu(isProd: isProd, isModal: isModal, cid: "회원관리번호", gender: "M", birthYear: "1996")
```

* Objective-C
```
BOOL isProd = YES(상용) or NO(개발);
BOOL isModal = YES(모달) or NO(스택);
[TASDKManager openLabangEarningMenuWithIsProd:isProd isModal:isModal cid:@"회원관리번호" gender:@"M" birthYear:@"1996"];
```


## 빌드시  주의사항

* 애플 앱스토어 혹은 TestFlight 를 통한 앱배포시에는 x86_64 아키텍쳐 빌드가 제외된 SDK 로 빌드하여야 합니다.
* arm64, x86_64 빌드 SDK :  폴더/ios_touchAd_sdk/TouchadSDK.xcframework

## Sample 프로젝트

* 프로젝트명 : ios_touchAd
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.

