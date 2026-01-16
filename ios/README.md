#  TouchAd SDK  for SKT 설치 가이드

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
platform :ios, '15.0'
use_frameworks!

target 'TouchadSDK' do

  pod 'SnapKit', '~> 5.7.1'
  pod 'Alamofire', '~> 5.9.0'
  pod 'ObjectMapper', '~> 4.2.0'
  pod 'JWTDecode', '~> 2.5.0'
  
end
post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['BUILD_LIBRARY_FOR_DISTRIBUTION'] = 'YES'
            config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '15.0'
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

## 터치애드 플랫폼 클래스 함수

- 주요기능을 모듈화하여 Static 함수형태로 호출합니다.
- 아래 간략한 설명입니다.
```
public class TASDKManager: NSObject {

/**
* 참여적립 화면 시작
* @param mbrId: MP 멤버십카드번호 (필수)
* @param adPushYn: MP 광고푸시수신여부 Y,N (필수)
* @param gender: MP 멤버십회원 성별 1(남),2(여),3(2000년이후 남),4(2000년이후 여) (필수)
* @param birthYear: MP 멤버십회원 생년월일 YYMMDD 6자리 (필수)
*/
func openMPEarningMenu(_ mbrId : String, adPushYn : String, gender : String, birthYear : String)

/**
* 돈 버는 교통 화면 시작
* @param mbrId: MP 멤버십카드번호 (필수)
* @param adPushYn: MP 광고푸시수신여부 Y,N (필수)
* @param gender: MP 멤버십회원 성별 1(남),2(여),3(2000년이후 남),4(2000년이후 여) (필수)
* @param birthYear: MP 멤버십회원 생년월일 YYMMDD 6자리 (필수)
* @param callback: MP 설정화면 오픈함수 (옵션)
*/
func openMPTouchadMenu(_ mbrId : String, adPushYn : String, gender : String, birthYear : String, callback: (() -> Void)?)

/**
* 돈 버는 교통 전면광고 오픈
* @param mbrId: MP 멤버십카드번호 (필수)
* @param userInfo: apns custom data
*/
func openMPAdvertise(_ mbrId : String, userInfo: [AnyHashable : Any])

/**
* 터치애드 참여적립 2차푸시 결과 화면
* @param mbrId: MP 멤버십카드번호 (필수)
* @param userInfo: apns custom data
*/
func openMPEarningResult(_ mbrId : String, userInfo: [AnyHashable : Any])

/**
* 오늘의 적립 배너 화면 시작
* @param mbrId: MP 멤버십카드번호 (필수)
* @param adPushYn: MP 광고푸시수신여부 Y,N (필수)
* @param gender: MP 멤버십회원 성별 1(남),2(여),3(2000년이후 남),4(2000년이후 여) (필수)
* @param birthYear: MP 멤버십회원 생년월일 YYMMDD 6자리 (필수)
*/
func openMPBanner(_ mbrId : String, adPushYn : String, gender : String, birthYear : String)

}
```

~~## 터치애드 초기화~~


## 돈 버는 교통 전면광고 화면 시작 (백그라운드, IOS >= 10)

*  MP 지하철 교통카드 승하차 푸시 수신하고 이때 돈 버는 교통 전면광고 화면을 띄울 경우 호출합니다.

*  MP 앱이 미실행 상태이거나 백그라운드 상태일 경우 MP앱이 실행된후에 전면광고 화면이 나타납니다.

* 아래는 돈 버는 교통 전면광고 시작함수 호출 예시입니다.
```
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        
    TASDKManager.openMPAdvertise("멤버십카드번호", userInfo: response.notification.request.content.userInfo)
        
    completionHandler()
}
```

## 돈 버는 교통 전면광고 화면 시작 (포그라운드, IOS >= 10)

*  MP 지하철 교통카드 승하차 푸시 수신하고 이때 돈 버는 교통 전면광고 화면을 띄울 경우 호출합니다.

*  MP 앱이 실행 상태일 경우 전면광고 화면이 나타납니다.

* 아래는 돈 버는 교통 전면광고 시작함수 호출 예시입니다.
```
func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    printd("willPresentNotification = \(notification.request.content.userInfo)")
    
    TASDKManager.openMPAdvertise("멤버십카드번호", userInfo: notification.request.content.userInfo)
    
    completionHandler([.alert, .badge, .sound])
}
```

## 돈 버는 교통 전면광고 화면 시작 (IOS < 10)

*  MP 지하철 교통카드 승하차 푸시 수신하고 이때 돈 버는 교통 전면광고 화면을 띄울 경우 호출합니다.

* 아래는 돈 버는 교통 전면광고 시작함수 호출 예시입니다.
```
private func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject]) {
    //[AnyHashable : Any]
    
    TASDKManager.openMPAdvertise("멤버십카드번호", userInfo: userInfo)

}
```

~~## 터치애드 광고 플랫폼 회원처리 시작~~


## 참여적립 화면 시작

*  MP 앱 내에서 참여적립 메뉴를 선택하면 약관동의 거치고 참여적립 화면을 시작할때 호출합니다.

* 아래는 참여적립 화면 시작함수 호출 예시입니다.
```
TASDKManager.openMPEarningMenu("멤버십카드번호",adPushYn:"Y", gender: "2", birthYear: "010915")
```

## 돈 버는 교통 화면 시작

*  MP 앱 내에서 돈 버는 교통 메뉴를 선택하면 약관동의 거치고 돈 버는 교통 화면을 시작할때 호출합니다.

*  MP 앱 광고푸시 설정 화면을 오픈할수 있는 기능을 콜백 영역내 구현합니다.  (옵션)

*  아래는 돈 버는 교통 화면 시작함수 호출 예시입니다.
```
TASDKManager.openMPTouchadMenu("멤버십카드번호",adPushYn:"Y", gender: "2", birthYear: "010915", callback:{() in 
 //MP 앱내 광고푸시 설정화면 오픈
})
```

## 참여적립 2차 푸시 결과 화면 (백그라운드, IOS >= 10)

*  참여적립 메뉴에서 광고 이용후 2차푸시(적립결과)를 수신하고 이때 참여적립 메인화면과 알림창을 띄울 경우 호출합니다. 

*  MP 앱이 미실행 상태이거나 백그라운드 상태일 경우 MP앱이 실행된후에 화면이 나타납니다.

*  아래는 참여적립 2차 푸시 결과 화면 함수 호출 예시입니다.
```
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    
    //userInfo에 dalcoin 데이터 존재하고 platformId 데이터 'SKTE' 값일때 호출합니다.
    TASDKManager.openMPEarningResult("멤버십카드번호", userInfo: response.notification.request.content.userInfo)
        
    completionHandler()
}
```

## 참여적립 2차 푸시 결과 화면 (포그라운드, IOS >= 10)

*  참여적립 메뉴에서 광고 이용후 2차푸시(적립결과)를 수신하고 이때 참여적립 메인화면과 알림창을 띄울 경우 호출합니다. 

*  MP 앱이 실행 상태일 경우 전면광고 화면이 나타납니다.

*  아래는 참여적립 2차 푸시 결과 화면 함수 호출 예시입니다.
```
func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    printd("willPresentNotification = \(notification.request.content.userInfo)")
    
    //userInfo에 dalcoin 데이터 존재하고 platformId 데이터 'SKTE' 값일때 호출합니다.
    TASDKManager.openMPEarningResult("멤버십카드번호", userInfo: notification.request.content.userInfo)
    
    completionHandler([.alert, .badge, .sound])
}
```

## 참여적립 2차 푸시 결과 화면 (IOS < 10)

*  참여적립 메뉴에서 광고 이용후 2차푸시(적립결과)를 수신하고 이때 참여적립 메인화면과 알림창을 띄울 경우 호출합니다. 

*  아래는 참여적립 2차 푸시 결과 화면 함수 호출 예시입니다.
```
private func application(application: UIApplication, didReceiveRemoteNotification userInfo: [NSObject : AnyObject]) {
    //[AnyHashable : Any]
    
    //userInfo에 dalcoin 데이터 존재하고 platformId 데이터 'SKTE' 값일때 호출합니다.
    TASDKManager.openMPEarningResult("멤버십카드번호", userInfo: userInfo)

}
```

## 오늘의 적립 배너 화면 시작

*  MP 앱 내에서 오늘의 적립 배너를 터치시 약관동의 거치고 오늘의 적립 배너 화면을 시작할때 호출합니다.

* 아래는 오늘의 적립 배너 화면 시작함수 호출 예시입니다.
```
TASDKManager.openMPBanner("멤버십카드번호",adPushYn:"Y", gender: "2", birthYear: "010915")
```


## 돈 버는 교통 교통카드 등록

* 돈 버는 교통 화면 메인 상단에 교통카드 등록 이미지를 터치시 교통카드 등록 화면으로 이동합니다.


## FCM 전송

* 터치애드는 푸시 송신 시 MP 에서 제공한 Public API를 이용하여 Push(FCM)를 전송합니다.
* Form파라미터(**필수**)

| 파라미터 | 내용 |
|---|---|
| `touchad`|문자열|

* API를 통해 Post된 데이터를 FCM데이터 구성요소 중 data와 payload 프로퍼티에 담아서 FCM전송 바랍니다.(※ 변경 가능성 있습니다.)

* FCM 전송 포맷 예시
```
{
  "android": {
    "priority": "high",
    "data": {
      "touchad": "%7B%22touchad%22%3A%22touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fskt%3FonOff%3D1%26cd%3D125%26cardIdx%3D1565%26areaCd%3DBSUB%22%7D"
    }
  },
  "apns": {
    "headers": {
      "apns-priority": "10"
    },
    "payload": {
      "aps": {
        "alert": {
          "title": "터치애드",
          "subtitle": "광고시청하면 리브포인트 적립!!!",
          "body": "출퇴근시간 지하철 통신비 절약되는 리브광고 많이 이용해주세요."
        },
        "category": "EVENT_INVITATION"
      },
      "touchad": "%7B%22touchad%22%3A%22touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fskt%3FonOff%3D1%26cd%3D125%26cardIdx%3D1565%26areaCd%3DBSUB%22%7D"
    },
    "fcm_options": {
      "image": "https://t.ta.runcomm.co.kr/html/img/profile00.png"
    }
  },
  "tokens": [
    "f3T_OObOQX-yo4J3y5bjcG:APA91bGzI2k8Fiz41ivql0ZV10hXLJz7w11Ne5Nf9IiZ1FymlJcGi-QRzv2lg3k46AYKamx-va2dyzj7m6TJTfCSTzTuPA7chomgSO_7PIh4LjsJ33SP7pDUoPvlGOeiM6oi5YXLiGvL"
  ]
}
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

