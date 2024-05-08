#  TouchAd SDK  for NH멤버스 설치 가이드

* NH멤버스 광고플랫폼 제휴서비스를 위한 NH터치애드 SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 TouchadSDK.framework 폴더를 프로젝트 소스폴더내 적절히 위치시켜 줍니다.
* 앱프로젝트 target > general > Frameworks,Libraries, and Embedded Content 에서 add files 에서 TouchadSDK.framework폴더를 선택합니다.
* Frameworks,Libraries, and Embedded Content 메뉴에서 TouchadSDK.framework의 Embed 옵션을 ‘Embed & Sign’ 선택합니다.
* Xcode 15.1 Build, Minimum Deployment 13.0 입니다.


## CocoaPods 설정
1. **Podfile 파일수정**
* SDK에서 사용하는 cocoapod 라이브러리 입니다.
* 프로젝트 Podfile 에 아래내용을 추가합니다.
* config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0' 설정은 Xcode 14.3 업데이트 후 앱 빌드 시 필요한 조건입니다.
```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '13.0'
use_frameworks!

target 'TouchadSDK' do

  pod 'Alamofire', '~> 5.9.0'
  pod 'ObjectMapper', '~> 4.2.0'
  pod 'JWTDecode', '~> 2.5.0'
  
end

post_install do |installer|
    installer.generated_projects.each do |project|
          project.targets.each do |target|
              target.build_configurations.each do |config|
                  config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
               end
          end
   end
end
```

## 권한 설정

1. **광고식별자(IDFA)**
* NH터치애드는 IDFA 값을 사용하여 사용자의 광고 사용 트래킹을 합니다.  
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

## NH터치애드 플랫폼 public 클래스

- 주요기능화면을 제어할 수 있게 UIViewController 확장클래스를 이용합니다.
- 아래 간략한 설명입니다.
```
/**
* NH터치애드 ViewController
* 생성자 매개변수
* @param isProd: 개발 / 상용 도메인을 설정하는 Bool 값(필수 값, true = 상용 도메인, false = 개발 도메인)
* @param cid: NH멤버스 회원관리번호 (필수)
*/
@objc public class NHEarningMenuViewController: UINavigationController

/**
* NH띠링 전면광고 ViewController
* 생성자 매개변수
* @param isProd: 개발 / 상용 도메인을 설정하는 Bool 값(필수 값, true = 상용 도메인, false = 개발 도메인)
* @param cid: NH멤버스 회원관리번호 (필수)
* @param userInfoString: apns custom data 문자열(필수)
* CommonWebViewController : UIViewController를 상속받는 확장 클래스
*/
@objc public class NHAdvertiseViewController: CommonWebViewController

```


## NH띠링 전면광고 화면 시작 (백그라운드, IOS >= 10)

* NH멤버스내 포인트 사용/적립 후 푸시 수신하고 터치 시 NH띠링의 전면광고 화면을 띄울 경우 호출합니다.

* 전면광고 화면이 올라온 상태에서 푸시 수신하고 터치 시 addUserInfoWithString함수에 userInfoString을 넣어 호출하면 됩니다.

* NH터치애드 화면이 올라온 상태에서 푸시 수신하고 터치 시 NH터치애드 화면을 종료한 뒤 전면광고 화면을 띄우면 됩니다.

* NH멤버스 앱이 미실행 상태이거나 백그라운드 상태일 경우 NH멤버스 앱이 실행된후에 전면광고 화면이 나타납니다.

* 아래는 NH띠링 전면광고 ViewController 호출 예시입니다.

* Swift
```
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    
    let isProd : Bool = true(상용 도메인) 또는 false(개발 도메인)

    let data : String = response.notification.request.content.userInfo["touchad"])

    //NH멤버스 화면일 경우
    if let nhmvc = self.window?.visibleViewController as? NH멤버스ViewController
    {
        let controller = NHAdvertiseViewController(isProd: isProd, cid: "회원관리번호", userInfoString: data)
        nhmvc.present(controller, animated:true, completion: nil)
    }
    //NH띠링 전면광고 화면일 경우
    else if let navc = self.window?.visibleViewController?.navigationController as? NHAdvertiseViewController 
    {
        navc.addUserInfoWithString(userInfoString: data)
    }
    //NH터치애드 화면일 경우
    else if let nevc = self.window?.visibleViewController?.navigationController as? NHEarningMenuViewController
    {
        nevc.dismiss(animated: false) {
            if let nhmvc = self.window?.visibleViewController as? NH멤버스ViewController {
                let controller = NHAdvertiseViewController(isProd: self.isProd, cid: "회원관리번호", userInfoString: data)
                nhmvc.present(controller, animated:true, completion: nil)
            }
        }
    }
    
    completionHandler()
}
```

* Objective-C
```
- (void)userNotificationCenter:(UNUserNotificationCenter *)center
        didReceiveNotificationResponse:(UNNotificationResponse *)response
         withCompletionHandler:(void (^)(void))completionHandler
{
    BOOL isProd = YES(상용 도메인) 또는 NO(개발 도메인);
    NSString *data = response.notification.request.content.userInfo[@"touchad"];
    ViewController* mvc = [AppDelegate visibleViewController];
    NHAdvertiseViewController* navc = [AppDelegate visibleViewController].navigationController;
    NHEarningMenuViewController* nevc = [AppDelegate visibleViewController].navigationController;
    
    //NH멤버스 화면일 경우
    if (mvc != nil)
    {
        NHAdvertiseViewController* vc = [[NHAdvertiseViewController alloc] initWithIsProd: isProd cid:@"회원관리번호"
        userInfoString:data];
        [mvc presentViewController:vc animated:YES completion:nil];
    }
    //전면광고 화면일 경우
    else if (navc != nil)
    {
        [navc addUserInfoWithStringWithUserInfoString: data];
    }
    //NH터치애드 화면일 경우
    else if (nevc != nil)
    {
        [nevc dismissViewControllerAnimated: NO completion:^{
            NHAdvertiseViewController* vc = [[NHAdvertiseViewController alloc] initWithIsProd: isProd cid:@"회원관리번호"
            userInfoString:data];
            [mvc presentViewController:vc animated:YES completion:nil];
        }];
    }
}

+ (UIViewController*) visibleViewController
{
    UIViewController *topController = [UIApplication sharedApplication].keyWindow.rootViewController;

    while (topController.presentedViewController) {
        topController = topController.presentedViewController;
    }

    return topController;
}
```

## NH띠링 전면광고 화면 시작 (포그라운드, IOS >= 10)

* NH멤버스내 포인트 사용/적립 후 푸시 수신하고 터치시 NH띠링의 전면광고 화면을 띄울 경우 호출합니다.

* NH멤버스 앱이 실행 상태일 경우 푸시 터치 시 전면광고 화면이 바로 나타납니다.

* 아래는 NH띠링 전면광고 ViewController 호출 예시입니다.

* Swift
```
func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
    printd("willPresentNotification = \(notification.request.content.userInfo)")
    
    completionHandler([.alert, .badge, .sound])
}
```

* Objective-C
```
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification
         withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler{
    
    completionHandler(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge);

};
```

## NH띠링 전면광고 ViewController 파라미터 userInfoString 전달형식

* NH띠링 전면광고 시작함수 호출 시 userInfoString을 아래 예시와 같은 형식으로 JSON String 전체를 전달하면됩니다.

* Swift
```
let isProd : Bool = true(상용 도메인) 또는 false(개발 도메인)
    
let userInfoString: String = 
"{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2F1.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"

let controller = NHAdvertiseViewController(isProd: isProd, cid: "회원관리번호", userInfoString: userInfoString)
self.navigationController?.pushViewController(controller, animated: true)
```

* Objective-C
```
BOOL isProd = YES(상용 도메인) 또는 NO(개발 도메인)

NSString* useInfo = [[NSString alloc] initWithString:@"{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2F1.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"];

NHAdvertiseViewController* vc = [[NHAdvertiseViewController alloc] initWithIsProd: isProd cid:@"회원관리번호" userInfoString:useInfo];
[self.navigationController pushViewController:vc animated:YES completion:nil];
```

## NH터치애드 화면 시작

* NH멤버스 앱 내에서 NH터치애드 메뉴를 선택하면 약관동의 거치고 NH터치애드 화면을 시작할때 호출합니다.

* 아래는 NH터치애드 ViewController 호출 예시입니다.

* Swift
```
let isProd : Bool = true(상용 도메인) 또는 false(개발도메인)

let navController = NHEarningMenuViewController(isProd: isProd, cid: "회원관리번호")
self.present(navController, animated:true, completion: nil)
```

* Objective-C
```
BOOL isProd = YES(상용 도메인) 또는 NO(개발 도메인)

NHEarningMenuViewController* vc = [[NHEarningMenuViewController alloc] initWithIsProd: isProd cid:@"회원관리번호"];
[self.navigationController presentViewController:vc animated:YES completion:nil];
```

## FCM 전송

* NH띠링은 푸시 송신 시 NH멤버스 에서 제공한 Public API를 이용하여 Push(FCM)를 전송합니다.
* Form파라미터(**필수**)

| 파라미터 | 내용 |
|---|---|
| `touchad`|문자열|

* API를 통해 Post된 데이터를 FCM데이터 구성요소 중 data와 payload 프로퍼티에 담아서 FCM전송 바랍니다.(※ 변경 가능성 있습니다.)

* FCM 전송 포맷 예시
```
개발 도메인 : t.ta.runcomm.co.kr
상용 도메인 : 1.ta.runcomm.co.kr
{
  "android": {
    "priority": "high",
    "data": {
      "touchad": "{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2F1.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"
    }
  },
  "apns": {
    "headers": {
      "apns-priority": "10"
    },
    "payload": {
      "aps": {
        "alert": {
          "title": "NH띠링",
          "subtitle": "광고시청하면 포인트 적립!!!",
          "body": "출퇴근시간 지하철 통신비 절약되는 NH멤버스 광고 많이 이용해주세요."
        },
        "category": "EVENT_INVITATION"
      },
      "touchad": 
      "{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2F1.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"
    },
    "fcm_options": {
      "image": "https://1.ta.runcomm.co.kr/html/img/profile00.png" 
    }
  },
  "tokens": [
    "f3T_OObOQX-yo4J3y5bjcG:APA91bGzI2k8Fiz41ivql0ZV10hXLJz7w11Ne5Nf9IiZ1FymlJcGi-QRzv2lg3k46AYKamx-va2dyzj7m6TJTfCSTzTuPA7chomgSO_7PIh4LjsJ33SP7pDUoPvlGOeiM6oi5YXLiGvL"
  ]
}
```

## 빌드시  주의사항

* 애플 앱스토어 혹은 TestFlight 를 통한 앱배포시에는 x86_64 아키텍쳐 빌드가 제외된 SDK 로 빌드하여야 합니다.
* arm64  빌드 SDK :  폴더/ios_touchAd_sdk/배포용/TouchadSDK.framework

## Sample 프로젝트

* 프로젝트명 : ios_touchAd
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
