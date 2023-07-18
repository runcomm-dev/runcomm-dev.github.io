#  TouchAd SDK  for BC 페이북 설치 가이드

* 정상적인 제휴서비스를 위한 터치애드SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 TouchadSDK.xcframework 폴더를 프로젝트 소스폴더내 적절히 위치시켜 줍니다.
* 앱프로젝트 target > general > Frameworks,Libraries, and Embedded Content 에서 add files 에서 TouchadSDK.xcframework폴더를 선택합니다.
* Frameworks,Libraries, and Embedded Content 메뉴에서 TouchadSDK.xcframework의 Embed 옵션을 ‘Embed & Sign’ 선택합니다.


## CocoaPods 설정
1. **Podfile 파일수정**
* SDK에서 사용하는 cocoapod 라이브러리 입니다.
* 프로젝트 Podfile 에 아래내용을 추가합니다.
```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '11.0'

use_frameworks!

target 'TouchadSDK' do

  pod 'SnapKit', '~> 4.0.1'
  pod 'Alamofire', '~> 4.8.2'
  pod 'ObjectMapper', '~> 4.2.0'
  pod 'JWTDecode', '~> 2.5.0'
end
post_install do |installer|
    installer.generated_projects.each do |project|
        project.targets.each do |target|
            target.build_configurations.each do |config|
                config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '11.0'
            end
        end
    end
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'YES'
        end
    end
end
```

## 권한 설정
1. **광고식별자(IDFA)**
* 터치애드는 IDFA 값을 사용하여 사용자의 광고 사용 트래킹을 합니다.  
* IOS 14 이상부터 IDFA 를 사용하기 위해선 명시적으로 사용자 동의를 얻어야 합니다.
* 시뮬레이터로 동작 시 IDFA가 고정값으로 적용됩니다.(시뮬레이터에서 IDFA를 가져올 경우 값이 0으로 표현되어 화면진입이 되지 않는 문제 대응)
* 시뮬레이터용 IDFA 고정값 : aabc1234-a12b-a123-abcd-abcde1234567
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
public class TASDKManager: NSObject {

/**
* 플러스적립 화면 시작(머니박스 당첨화면)
* @param mbrId: BC 페이북 머니회원번호 (필수)
*/
func openBCPlusMoneyMenu(_ mbrId : String)

/**
* 플러스적립 화면 시작(출석체크 클로징 배너)
* @param mbrId: BC 페이북 머니회원번호 (필수)
*/
func openBCPlusBannerMenu(_ mbrId : String)

/**
* 플러스적립 화면 시작(출석체크 메인 화면)
* @param mbrId: BC 페이북 머니회원번호 (필수)
*/
func openBCPlusMainMenu(_ mbrId : String)

}
```

~~## 터치애드 광고 플랫폼 회원처리 시작~~


## 플러스적립 화면 시작(머니박스 당첨화면)

*  BC 페이북 앱 내 머니박스 당첨 화면 팝업 화면에서 '머니박스 3개 더 받기' 버튼을 터치시 호출합니다.

*  아래는 딥링크를 통해 호출하는 플러스적립 화면 시작함수 예시입니다.
```
딥링크 - app://plusmoney?mbrId={머니회원번호}

함수호출 - TASDKManager.openBCPlusMoneyMenu("머니회원번호")
```

## 플러스적립 화면 시작(출석체크 클로징 배너)

*  BC 페이북 앱 내 출석체크 클로징 배너에서 '머니 PLUS' 버튼을 터치시 호출합니다.

*  아래는 딥링크를 통해 호출하는 플러스적립 화면 시작함수 예시입니다.
```
딥링크 - app://plusbanner?mbrId={머니회원번호}

함수호출 - TASDKManager.openBCPlusBannerMenu("머니회원번호")
```

## 플러스적립 화면 시작(출석체크 메인 화면)

*  BC 페이북 앱 내 출석체크 메인 화면에서 플러스적립 화면 시작 버튼을 터치시 호출합니다.

*  아래는 딥링크를 통해 호출하는 플러스적립 화면 시작함수 예시입니다.
```
딥링크 - app://plusmain?mbrId={머니회원번호}

함수호출 - TASDKManager.openBCPlusMainMenu("머니회원번호")
```


## 빌드시  주의사항

* 애플 앱스토어 혹은 TestFlight 를 통한 앱배포시에는 x86_64 아키텍쳐 빌드가 제외된 SDK 로 빌드하여야 합니다.
* arm64  빌드 SDK :  폴더/ios_touchAd_sdk/배포용/TouchadSDK.xcframework
* XCode 에뮬레이터를 이용한 앱 개발시에는 x86_64 아키텍쳐 빌드가 포함된 SDK 로 빌드하여야 합니다.
* arm64, x86_64 빌드 SDK : 폴더/ios_touchAd_sdk/개발용/TouchadSDK.xcframework

## Sample 프로젝트

* 프로젝트명 : ios_touchAd
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.

