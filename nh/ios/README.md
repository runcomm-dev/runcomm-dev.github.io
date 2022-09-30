#  TouchAd SDK  for NH멤버스 설치 가이드

* NH멤버스 광고플랫폼 제휴서비스를 위한 NH터치애드 SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 TouchadSDK.framework 폴더를 프로젝트 소스폴더내 적절히 위치시켜 줍니다.
* 앱프로젝트 target > general > Frameworks,Libraries, and Embedded Content 에서 add files 에서 TouchadSDK.framework폴더를 선택합니다.
* Frameworks,Libraries, and Embedded Content 메뉴에서 TouchadSDK.framework의 Embed 옵션을 ‘Embed & Sign’ 선택합니다.


## CocoaPods 설정
1. **Podfile 파일수정**
* SDK에서 사용하는 cocoapod 라이브러리 입니다.
* 프로젝트 Podfile 에 아래내용을 추가합니다.
```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '10.0'
use_frameworks!

target 'TouchadSDK' do

  pod 'SnapKit', '~> 4.0.1'
  pod 'Alamofire', '~> 4.8.2'
  pod 'ObjectMapper', '~> 4.2.0'
  pod 'JWTDecode', '~> 2.5.0'
  
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

## NH터치애드 플랫폼 public 클래스

- 주요기능화면을 제어할 수 있게 UIViewController 확장클래스를 이용합니다.
- 아래 간략한 설명입니다.
```
/**
* NH터치애드 ViewController
* 생성자 매개변수
* @param cid: NH멤버스 회원관리번호 (필수)
*/
@objc public class NHEarningMenuViewController: UINavigationController

/**
* NH띠링 전면광고 ViewController
* 생성자 매개변수
* @param cid: NH멤버스 회원관리번호 (필수)
* @param userInfoString: apns custom data 문자열(필수)
*/
@objc public class NHAdvertiseViewController: UINavigationController

/**
* 적립문의 ViewController
* 생성자 매개변수
* @param cid: NH멤버스 회원관리번호 (필수)
*/
@objc public class NHInquiryMenuViewController: UINavigationController

/**
* 이용안내 ViewController
*/
@objc public class NHUseInfoMenuViewController: UINavigationController

/**
* 공지사항 ViewController
*/
@objc public class NHNoticeMenuViewController: UINavigationController

/**
* 참여이력 ViewController
* 생성자 매개변수
* @param cid: NH멤버스 회원관리번호 (필수)
*/
@objc public class NHApprlNoMenuViewController: UINavigationController

```


## NH띠링 전면광고 화면 시작 (백그라운드, IOS >= 10)

* NH멤버스내 포인트 사용/적립 후 푸시 수신하고 터치시 NH띠링의 전면광고 화면을 띄울 경우 호출합니다.

* NH멤버스 앱이 미실행 상태이거나 백그라운드 상태일 경우 NH멤버스 앱이 실행된후에 전면광고 화면이 나타납니다.

* 아래는 NH띠링 전면광고 ViewController 호출 예시입니다.

* Swift
```
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        
    if let vc = window?.rootViewController as? UINavigationController {
        let navController = NHAdvertiseViewController(cid: "회원관리번호", userInfoString: response.notification.request.content.userInfo["touchad"])
        vc.present(navController, animated:true, completion: nil)
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
        NHAdvertiseViewController* vc = [[NHAdvertiseViewController alloc] initWithCid:@"회원관리번호" 
        userInfoString:response.notification.request.content.userInfo["touchad"]];
        [self.navigationController presentViewController:vc animated:YES completion:nil];
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
let userInfoString: String = 
"{\"cid\":\"poqwer2886\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"

let navController = NHAdvertiseViewController(cid: "회원관리번호", userInfoString: userInfoString)
self.present(navController, animated:true, completion: nil)
```

* Objective-C
```
NSString* useInfo = [[NSString alloc] initWithString:@"{\"cid\":\"poqwer2886\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"];

NHAdvertiseViewController* vc = [[NHAdvertiseViewController alloc] initWithCid:@"회원관리번호" userInfoString:useInfo];
[self.navigationController presentViewController:vc animated:YES completion:nil];
```

## NH터치애드 화면 시작

* NH멤버스 앱 내에서 NH터치애드 메뉴를 선택하면 약관동의 거치고 NH터치애드 화면을 시작할때 호출합니다.

* 아래는 NH터치애드 ViewController 호출 예시입니다.

* Swift
```
let navController = NHEarningMenuViewController(cid: "회원관리번호")
self.present(navController, animated:true, completion: nil)
```

* Objective-C
```
NHEarningMenuViewController* vc = [[NHEarningMenuViewController alloc] initWithCid:@"회원관리번호"];
[self.navigationController presentViewController:vc animated:YES completion:nil];
```

## 적립문의 화면 시작

*  NH멤버스 앱 내에서 적립문의 메뉴를 선택하면 적립문의 화면을 시작할때 호출합니다.

*  아래는 적립문의 ViewController 호출 예시입니다.

* Swift
```
let navController = NHInquiryMenuViewController(cid: "회원관리번호")
self.present(navController, animated:true, completion: nil)
```

* Objective-C
```
NHInquiryMenuViewController* vc = [[NHInquiryMenuViewController alloc] initWithCid:@"회원관리번호"];
[self.navigationController presentViewController:vc animated:YES completion:nil];
```

## 이용안내 화면 시작

*  NH멤버스 앱 내에서 이용안내 메뉴를 선택하면 이용안내 화면을 시작할때 호출합니다.

*  아래는 이용안내 ViewController 호출 예시입니다.

* Swift
```
let navController = NHUseInfoMenuViewController()
self.present(navController, animated:true, completion: nil)
```

* Objective-C
```
NHUseInfoMenuViewController* vc = [[NHUseInfoMenuViewController alloc] init];
[self.navigationController presentViewController:vc animated:YES completion:nil];
```

## 공지사항 화면 시작

*  NH멤버스 앱 내에서 공지사항 메뉴를 선택하면 공지사항 화면을 시작할때 호출합니다.

*  아래는 공지사항 ViewController 호출 예시입니다.

* Swift
```
let navController = NHNoticeMenuViewController()
self.present(navController, animated:true, completion: nil)
```

* Objective-C
```
NHNoticeMenuViewController* vc = [[NHNoticeMenuViewController alloc] init];
[self.navigationController presentViewController:vc animated:YES completion:nil];
```

## 참여이력 화면 시작

*  NH멤버스 앱 내에서 참여이력 메뉴를 선택하면 참여이력 화면을 시작할때 호출합니다.

*  아래는 참여이력 ViewController 호출 예시입니다.

* Swift
```
let navController = NHApprlNoMenuViewController(cid: "회원관리번호")
self.present(navController, animated:true, completion: nil)
```

* Objective-C
```
NHApprlNoMenuViewController* vc = [[NHApprlNoMenuViewController alloc] initWithCid:@"회원관리번호"];
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
{
  "android": {
    "priority": "high",
    "data": {
      "touchad": "{\"cid\":\"poqwer2886\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"
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
      "{\"cid\":\"poqwer2886\",\"apprlNo\":\"1\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",\"custom-type\":\"touchad\",\"custom-body\":\"%7B%22touchad%22%3A%22touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fnh%3FapprlNo%3D12345678%22%7D\",\"platformId\":\"NHJ\"}"
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
* armv7, arm64  빌드 SDK :  폴더/ios_touchAd_sdk/배포용/TouchadSDK.framework
* XCode 에뮬레이터를 이용한 앱 개발시에는 x86_64 아키텍쳐 빌드가 포함된 SDK 로 빌드하여야 합니다.
* armv7, arm64, x86_64 빌드 SDK : 폴더/ios_touchAd_sdk/개발용/TouchadSDK.framework

## Sample 프로젝트

* 프로젝트명 : ios_touchAd
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.


